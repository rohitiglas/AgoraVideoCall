# AgoraVideoCall

Step:1 

Add the following line in the /app/build.gradle file of your project:

dependencies {
    ...
    // 2.9.2 is the latest version of the Agora SDK. You can set it to other versions.
    implementation 'io.agora.rtc:full-sdk:2.9.2'
}

Step2:

Add project permissions
Add the following permissions in the /app/src/main/AndroidManifest.xml file for device access according to your needs:

<uses-permission android:name="android.permission.READ_PHONE_STATE" />   
   <uses-permission android:name="android.permission.INTERNET" />
   <uses-permission android:name="android.permission.RECORD_AUDIO" />
   <uses-permission android:name="android.permission.CAMERA" />
   <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
   <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
   <!-- The Agora SDK requires Bluetooth permissions in case users are using Bluetooth devices.-->
   <uses-permission android:name="android.permission.BLUETOOTH" />
   <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
   
   
   Step3: 
    Import Classes
Import the following classes in the activity file of your project:
import io.agora.rtc.IRtcEngineEventHandler;
import io.agora.rtc.RtcEngine;
import io.agora.rtc.video.VideoCanvas;

import io.agora.rtc.video.VideoEncoderConfiguration;

Step:4 
Get the device permission
Call the checkSelfPermission method to access the camera and the microphone of the Android device when launching the activity.

private static final int PERMISSION_REQ_ID = 22;

// Ask for Android device permissions at runtime.
private static final String[] REQUESTED_PERMISSIONS = {
        Manifest.permission.RECORD_AUDIO,
        Manifest.permission.CAMERA,
        Manifest.permission.WRITE_EXTERNAL_STORAGE
};

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_video_chat_view);

    // If all the permissions are granted, initialize the RtcEngine object and join a channel.
    if (checkSelfPermission(REQUESTED_PERMISSIONS[0], PERMISSION_REQ_ID) undefined
            checkSelfPermission(REQUESTED_PERMISSIONS[2], PERMISSION_REQ_ID)) {
        initEngineAndJoinChannel();
    }
}

private boolean checkSelfPermission(String permission, int requestCode) {
    if (ContextCompat.checkSelfPermission(this, permission) !=
            PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(this, REQUESTED_PERMISSIONS, requestCode);
        return false;
    }

    return true;
}


Step5:
Initialize RtcEngine
Create and initialize the RtcEngine object before calling any other Agora APIs.

In this step, you need to use the App ID of your project. Follow these steps to create an Agora project in Console and get an App ID.

Go to Console and click the Project Management icon  on the left navigation panel.
Click Create and follow the on-screen instructions to set the project name, choose an authentication mechanism, and Click Submit.
On the Project Management page, find the App ID of your project.
Call the create method and pass in the App ID to initialize the RtcEngine object.

You can also listen for callback events, such as when the local user joins the channel, and when the first video frame of a remote user is decoded. Do not implement UI operations in these callbacks.


private final IRtcEngineEventHandler mRtcEventHandler = new IRtcEngineEventHandler() {
    @Override
    // Listen for the onJoinChannelSuccess callback.
    // This callback occurs when the local user successfully joins the channel.
    public void onJoinChannelSuccess(String channel, final int uid, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","Join channel success, uid: " + (uid & 0xFFFFFFFFL));
            }
        });
    }

    @Override
    // Listen for the onFirstRemoteVideoDecoded callback.
    // This callback occurs when the first video frame of a remote user is received and decoded after the remote user successfully joins the channel.
    // You can call the setupRemoteVideo method in this callback to set up the remote video view.
    public void onFirstRemoteVideoDecoded(final int uid, int width, int height, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","First remote video decoded, uid: " + (uid & 0xFFFFFFFFL));
                setupRemoteVideo(uid);
            }
        });
    }

    @Override
    // Listen for the onUserOffline callback.
    // This callback occurs when the remote user leaves the channel or drops offline.
    public void onUserOffline(final int uid, int reason) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","User offline, uid: " + (uid & 0xFFFFFFFFL));
                onRemoteUserLeft();
            }
        });
    }
};

...

// Initialize the RtcEngine object.
private void initializeEngine() {
    try {
        mRtcEngine = RtcEngine.create(getBaseContext(), getString(R.string.agora_app_id), mRtcEventHandler);
    } catch (Exception e) {
        Log.e(TAG, Log.getStackTraceString(e));
        throw new RuntimeException("NEED TO check rtc sdk init fatal error\n" + Log.getStackTraceString(e));
    }
}


Step:6

Set the local video view
If you are implementing a voice call, skip to Join a channel.

After initializing the RtcEngine object, set the local video view before joining the channel so that you can see yourself in the call. Follow these steps to configure the local video view:

Call the enableVideo method to enable the video module.
Call the createRendererView method to create a SurfaceView object.
Call the setupLocalVideo method to configure the local video display settings.

private void setupLocalVideo() {

    // Enable the video module.
    mRtcEngine.enableVideo();

    // Create a SurfaceView object.
    private FrameLayout mLocalContainer;
    private SurfaceView mLocalView;

    mLocalView = RtcEngine.CreateRendererView(getBaseContext());
    mLocalView.setZOrderMediaOverlay(true);
    mLocalContainer.addView(mLocalView);
    // Set the local video view.
    VideoCanvas localVideoCanvas = new VideoCanvas(mLocalView, VideoCanvas.RENDER_MODE_HIDDEN, 0);
    mRtcEngine.setupLocalVideo(localVideoCanvas);
}

Step7: 

Join a channel
After initializing the RtcEngine object and setting the local video view (for a video call), you can call the joinChannel method to join a channel. In this method, set the following parameters:

token: Pass a token that identifies the role and privilege of the user. You can set it as one of the following values:

NULL.
A temporary token generated in Console. A temporary token is valid for 24 hours. For details, see Get a Temporary Token.
A token generated at the server. This applies to scenarios with high-security requirements. For details, see Generate a token from Your Server.
If your project has enabled the app certificate, ensure that you provide a token.
channelName: Specify the channel name that you want to join. Users that input the same channel name join the same channel.

uid: ID of the local user that is an integer and should be unique. If you set uid as 0, the SDK assigns a user ID for the local user and returns it in the onJoinChannelSuccess callback.

For more details on the parameter settings, see joinChannel.

private void joinChannel() {

    // Join a channel with a token.
    mRtcEngine.joinChannel(YOUR_TOKEN, "demoChannel1", "Extra Optional Data", 0);
}


Step8:
Set the remote video view
In a video call, you should be able to see other users too. This is achieved by calling the setupRemoteVideo method after joining the channel.

Shortly after a remote user joins the channel, the SDK gets the remote user's ID in the onFirstRemoteVideoDecoded callback. Call the setupRemoteVideo method in the callback, and pass in the uid to set the video view of the remote user.

    @Override
    // Listen for the onFirstRemoteVideoDecoded callback.
    // This callback occurs when the first video frame of a remote user is received and decoded after the remote user successfully joins the channel.
    // You can call the setupRemoteVideo method in this callback to set up the remote video view.
    public void onFirstRemoteVideoDecoded(final int uid, int width, int height, int elapsed) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                Log.i("agora","First remote video decoded, uid: " + (uid & 0xFFFFFFFFL));
                setupRemoteVideo(uid);
            }
        });
    }

private void setupRemoteVideo(int uid) {

    // Create a SurfaceView object.
    private RelativeLayout mRemoteContainer;
    private SurfaceView mRemoteView;


    mRemoteView = RtcEngine.CreateRendererView(getBaseContext());
    mRemoteContainer.addView(mRemoteView);
    // Set the remote video view.
    mRtcEngine.setupRemoteVideo(new VideoCanvas(mRemoteView, VideoCanvas.RENDER_MODE_HIDDEN, uid));

}

Step9:
Mute the local audio
Call the muteLocalAudioStream method to stop or resume sending the local audio stream to mute or unmute the local user.

public void onLocalAudioMuteClicked(View view) {
    mMuted = !mMuted;
    mRtcEngine.muteLocalAudioStream(mMuted);
}
Switch the camera direction
Call the switchCamera method to switch the direction of the camera.

public void onSwitchCameraClicked(View view) {
    mRtcEngine.switchCamera();
}

Step10:
Call the leaveChannel method to leave the current call according to your scenario, for example, when the call ends, when you need to close the app, or when your app runs in the background.

@Override
protected void onDestroy() {
    super.onDestroy();
    if (!mCallEnd) {
        leaveChannel();
    }
    RtcEngine.destroy();
}

private void leaveChannel() {
    // Leave the current channel.
    mRtcEngine.leaveChannel();
}
