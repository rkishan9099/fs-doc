# *React Native Sip.js video Call in Expo*

```bash
import  { useEffect, useState } from 'react';
import React = require('react');
import { View, Text, Button, TextInput, Alert, Platform, PermissionsAndroid } from 'react-native';
import { mediaDevices, MediaStream, RTCView } from 'react-native-webrtc';
import { Inviter, Registerer, RegistererState, SessionState, UserAgent, UserAgentOptions } from 'sip.js';

interface UserAgentOptionsType extends UserAgentOptions {
  dtmfType: string;
  contactTransport: string;
  keepAliveInterval: number;
  maxReconnectionAttempts: number;
  expires: number;
  transportOptions: {
    server: string;
    traceSip: boolean;
  };
}

type sipConfigType = {
  _logEnable: boolean;
  _iceServer: string;
}

const sipConfig: sipConfigType = {
  _logEnable: true,
  _iceServer: 'stun:stun.l.google.com:19302',
}

const SipjsVideoScreen = () => {
  const [ua, setUa] = useState<UserAgent | null>(null);
  const [localStream, setLocalStream] = useState<MediaStream | null>(null);
  const [remoteStream, setRemoteStream] = useState<MediaStream | null>(null);
  const [incomingSession, setIncomingSession] = useState<any | null>(null);
  const [username, setUsername] = useState<string>('7002');
  const [password, setPassword] = useState<string>('digit90digit@@digit90');
  const [destination, setDestination] = useState<string>('');
  const [webSocketUrl, setWebSocketUrl] = useState<string>('wss://s14switch.digitechnobytes.online:7443'); // Replace with your WebSocket URL

  const [callStatus, setCallStatus] = useState<string>('Disconnected');
  const [connected, setConnected] = useState<boolean>(false);

  // Request permissions for audio and video
  const requestPermissions = async () => {
    if (Platform.OS === 'android') {
      try {
        const granted = await PermissionsAndroid.requestMultiple(
          [PermissionsAndroid.PERMISSIONS.RECORD_AUDIO, PermissionsAndroid.PERMISSIONS.CAMERA]
        );
        if (
          granted[PermissionsAndroid.PERMISSIONS.RECORD_AUDIO] === PermissionsAndroid.RESULTS.GRANTED &&
          granted[PermissionsAndroid.PERMISSIONS.CAMERA] === PermissionsAndroid.RESULTS.GRANTED
        ) {
          console.log('Microphone and Camera permission granted');
        } else {
          console.log('Microphone and Camera permission denied');
        }
      } catch (err) {
        console.warn(err);
      }
    } else {
      console.log('Microphone and Camera permission is not required on iOS');
    }
  };

  useEffect(() => {
    requestPermissions();
  }, []);

  useEffect(() => {
    if (username && password && webSocketUrl) {
      const sip_aor = `sip:${username}@s14switch.digitechnobytes.online`;
      const userAgentOptions: UserAgentOptionsType = {
        uri: UserAgent.makeURI(sip_aor),
        transportOptions: {
          server: webSocketUrl,
          traceSip: true
        },
        userAgentString: username,
        authorizationPassword: password,
        authorizationUsername: username,
        dtmfType: 'info',
        contactTransport: 'wss',
        noAnswerTimeout: 60,
        displayName: username,
        contactParams: { transport: 'wss' },
        keepAliveInterval: 10,
        maxReconnectionAttempts: 3,
        expires: 3000,
        hackIpInContact: true,
        sessionDescriptionHandlerFactoryOptions: {
          peerConnectionOptions: {
            rtcpMuxPolicy: 'negotiate',
            iceCheckingTimeout: 1000,
            iceTransportPolicy: 'all',
            iceServers: [{ urls: sipConfig._iceServer }]
          }
        },
        logBuiltinEnabled: sipConfig._logEnable
      };

      const userAgent = new UserAgent(userAgentOptions);
      const registerer = new Registerer(userAgent, {});
      userAgent.start().then(() => {
        registerer.stateChange.addListener(newState => {
          switch (newState) {
            case RegistererState.Initial:
              console.log('UserAgent ==> Initial');
              break;
            case RegistererState.Registered:
              console.log('UserAgent ==> Registered');
              setConnected(true);
              break;
            case RegistererState.Unregistered:
              console.log('UserAgent ==> Unregistered');
              setConnected(false);
              break;
            case RegistererState.Terminated:
              console.warn('UserAgent ==> Terminated');
              setConnected(false);
              userAgent.stop();
              break;
            default:
              console.log('UserAgent ==> Unidentified');
              break;
          }
        });
        registerer
          .register()
          .then(() => {
            console.log('Successfully sent REGISTER');
          })
          .catch(error => {
            console.log('Failed to send REGISTER', error);
          });
      });
      setUa(userAgent);

      userAgent.delegate = {
        onInvite(invitation) {
          console.log('onInvite', invitation);
          setIncomingSession(invitation);
          invitation.accept().then(() => {
            handleIncomingSession(invitation);
          });
        },
      };

      return () => {
        if (userAgent) {
          userAgent.stop();
        }
      };
    }
  }, [username, password, webSocketUrl]);

  const handleIncomingSession = async (session: any) => {
    const pc = session.sessionDescriptionHandler.peerConnection;
    const localStream = await mediaDevices.getUserMedia({ audio: true, video: true });
    setLocalStream(localStream);
    localStream.getTracks().forEach((track: any) => pc.addTrack(track, localStream));

    const remoteStream = new MediaStream();
    pc.getReceivers().forEach((receiver: { track:any; }) => {
      remoteStream.addTrack(receiver.track);
    });
    setRemoteStream(remoteStream);

    session.stateChange.addListener((newState: any) => {
      setCallStatus(newState);
      switch (newState) {
        case SessionState.Establishing:
          console.debug('Session is establishing');
          break;
        case SessionState.Established:
          console.debug('Session established');
          break;
        case SessionState.Terminated:
          console.debug('Session terminated');
          break;
        default:
          break;
      }
    });
  };

  const makeCall = async () => {
    const uri: any = UserAgent.makeURI(`sip:${destination}@s14switch.digitechnobytes.online`);
    
    if (ua && destination) {
      const localStream = await mediaDevices.getUserMedia({ audio: true, video: true });
      setLocalStream(localStream);
      const inviteOptions = {
        sessionDescriptionHandlerOptions: {
          constraints: {
            audio: true,
            video: true
          },
        },
      };
      const session: any = new Inviter(ua, uri, inviteOptions);
      
      session.stateChange.addListener(async (newState: any) => {
        setCallStatus(newState);
        switch (newState) {
          case SessionState.Establishing:
            console.debug('Session is establishing');
            break;
          case SessionState.Established:
            if (session.sessionDescriptionHandler && session.sessionDescriptionHandler.peerConnection) {
              const pc = session.sessionDescriptionHandler.peerConnection;
              localStream.getTracks().forEach((track: any) => pc.addTrack(track, localStream));

              const remoteStream = new MediaStream();
              pc.getReceivers().forEach((receiver: { track: any; }) => {
                remoteStream.addTrack(receiver.track);
              });
              setRemoteStream(remoteStream);
            }
            break;
          case SessionState.Terminated:
            console.debug('Session terminated');
            break;
          default:
            break;
        }
      });
      session.invite().then(() => {
        console.log('Successfully sent INVITE');
      }).catch((error: any) => {
        console.log('Failed to send INVITE ==> ', error);
      });
    }
  };

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center', padding: 20 }}>
      <Text>SIP.js with Expo</Text>
      
      <TextInput
        style={{ height: 40, borderColor: 'gray', borderWidth: 1, marginVertical: 10, width: '100%', paddingHorizontal: 10 }}
        placeholder="SIP Username"
        value={username}
        onChangeText={setUsername}
      />
      
      <TextInput
        style={{ height: 40, borderColor: 'gray', borderWidth: 1, marginVertical: 10, width: '100%', paddingHorizontal: 10 }}
        placeholder="SIP Password"
        secureTextEntry
        value={password}
        onChangeText={setPassword}
      />
      
      <TextInput
        style={{ height: 40, borderColor: 'gray', borderWidth: 1, marginVertical: 10, width: '100%', paddingHorizontal: 10 }}
        placeholder="Destination"
        value={destination}
        onChangeText={setDestination}
      />
      
      <Button title="Make Call" onPress={makeCall} />
      
      {callStatus && <Text>Status: {callStatus}</Text>}
      
      <View style={{ marginVertical: 20 }}>
        {localStream && (
          <RTCView
            streamURL={localStream.toURL()}
            style={{ width: 100, height: 100 }}
            objectFit="cover"
          />
        )}
        {remoteStream && (
          <RTCView
            streamURL={remoteStream.toURL()}
            style={{ width: 100, height: 100 }}
            objectFit="cover"
          />
        )}
      </View>
    </View>
  );
};

export default SipjsVideoScreen;



```
