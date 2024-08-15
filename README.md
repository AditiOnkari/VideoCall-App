# VideoCall-App
import "./App.css";
import React, { useState, useEffect, useRef } from "react";

function VideoCallApp() {
  const [isCallEnabled, setIsCallEnabled] = useState(false);
  const [isRecording, setIsRecording] = useState(false);
  const [mediaRecorder, setMediaRecorder] = useState(null);
  const [recordedChunks, setRecordedChunks] = useState([]);

  const localVideoRef = useRef(null);
  const remoteVideoRef = useRef(null);
  const streamRef = useRef(null);
  const peerConnectionRef = useRef(null);

  useEffect(() => {
    const checkTime = () => {
      const now = new Date();
      const hours = now.getHours();
      if (hours >= 18 && hours < 24) {
        setIsCallEnabled(true);
      } else {
        setIsCallEnabled(false);
      }
    };

    checkTime();
    const interval = setInterval(checkTime, 60000); // Check every minute

    return () => clearInterval(interval);
  }, []);

  const startCall = async () => {
    try {
      const localStream = await navigator.mediaDevices.getUserMedia({
        video: true,
        audio: true,
      });
      localVideoRef.current.srcObject = localStream;
      streamRef.current = localStream;

      // Initialize WebRTC connection
      const peerConnection = new RTCPeerConnection();
      peerConnectionRef.current = peerConnection;

      localStream
        .getTracks()
        .forEach((track) => peerConnection.addTrack(track, localStream));

      // Handle remote stream
      peerConnection.ontrack = (event) => {
        remoteVideoRef.current.srcObject = event.streams[0];
      };

      // Offer/Answer exchange logic...

      // Prepare for recording
      const mediaRecorderInstance = new MediaRecorder(localStream);
      mediaRecorderInstance.ondataavailable = handleDataAvailable;
      setMediaRecorder(mediaRecorderInstance);
    } catch (err) {
      console.error("Error starting the call:", err);
    }
  };

  const handleDataAvailable = (event) => {
    if (event.data.size > 0) {
      setRecordedChunks((prev) => [...prev, event.data]);
    }
  };

  const startRecording = () => {
    mediaRecorder.start();
    setIsRecording(true);
  };

  const stopRecording = () => {  
    mediaRecorder.stop();
    setIsRecording(false);

    // Save the recorded video
    const blob = new Blob(recordedChunks, { type: "video/webm" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.style.display = "none";
    a.href = url;
    a.download = "recorded-session.webm";
    document.body.appendChild(a);
    a.click();
    window.URL.revokeObjectURL(url);
    setRecordedChunks([]);
  };

  const shareScreen = async () => {
    try {
      const screenStream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
      });
      screenStream
        .getTracks()
        .forEach((track) =>
          peerConnectionRef.current.addTrack(track, screenStream)
        );
      localVideoRef.current.srcObject = screenStream;
      streamRef.current = screenStream;
    } catch (err) {
      console.error("Error sharing screen:", err);
    }
  };

  return (
    <div>
      <h1>Video Call App</h1>
      <div>
        <video ref={localVideoRef} autoPlay muted />
        <video ref={remoteVideoRef} autoPlay />
      </div>
      <div>
        <button onClick={startCall} disabled={!isCallEnabled}>
          Start Call
        </button>
        <button onClick={shareScreen} disabled={!isCallEnabled}>
          Share Screen
        </button>
        <button
          onClick={startRecording}
          disabled={!isCallEnabled || isRecording}
        >
          Start Recording
        </button>
        <button
          onClick={stopRecording}
          disabled={!isCallEnabled || !isRecording}
        >
          Stop Recording
        </button>
      </div>
      {!isCallEnabled && (
        <p>Video calls are only available between 6 PM and 12 AM</p>
      )}
    </div>
  );
}

export default VideoCallApp;
 
