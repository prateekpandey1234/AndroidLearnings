## Android implementation :- (https://github.com/prateekpandey1234/AndroidStudy)
## Backend implementation :- (https://github.com/prateekpandey1234/PPsTREAMS/tree/master)


# Video Streaming 


1. Buffer is the process of preloading the data in temporary storage on device before running on player . so most whenever we are setting maximum buffer and minimum buffer in our video players that means :-

        a. maxBufferMs -> if the amount of video is Buffered already , then dont preload or prefetch any segments anymore .
        
        b. minBufferMs -> We need atleast this amount of video to be loaded to play the video .

2. Every live stream has it's manifest file (.m3u8 for HLS or .mpd for DASH) , it also shares important info about  video tracks , audio tracks and subtitles . it's more of a overview to the client what type of stream is it goingto be .

3. Below here is how a live stream works on lower scope :-
```
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:12345

#EXTINF:10.0,
segment_12345.ts
#EXTINF:10.0,
segment_12346.ts
#EXTINF:10.0,
segment_12347.ts
#EXTINF:10.0,
segment_12348.ts
...

**Each segment = 10 seconds of video/audio**

Now the **LoadControl** kicks in:
Initial Buffer Build:
├── Download segment_12345.ts (10s) 
├── Download segment_12346.ts (10s) ─┐
└── Total: 20s buffered              │
                                     │
    Player starts when 2s ready ◄────┘
    (bufferForPlaybackMs = 2_000)
```

**Parallel to playback:**
```
Timeline (seconds):
0s ──────────► Player starts playing segment_12345
               ↓
               While playing, downloads segment_12347
               ↓
10s ─────────► Switches to segment_12346
               ↓
               Downloads segment_12348
               ↓
20s ─────────► Switches to segment_12347
               ↓
               Downloads segment_12349
```

4. each segment is a MPEG transport stream , .ts file extension , the segment contain the data (auiod,video or text) , synchromisation pattern and error correction . these segments are made of different packets and each packet has header(type of stream) and payload . 
```
.ts file structure:
┌────────────────────────────────────────────┐
│ Transport Stream Container                 │
├────────────────────────────────────────────┤
│ ┌────────────────────────────────────┐     │
│ │ Packet 1 (188 bytes)               │     │
│ │ ├─ Header (4 bytes)                │     │
│ │ └─ Payload (184 bytes) → H.264     │     │
│ └────────────────────────────────────┘     │
│ ┌────────────────────────────────────┐     │
│ │ Packet 2 (188 bytes)               │     │
│ │ ├─ Header (4 bytes)                │     │
│ │ └─ Payload (184 bytes) → AAC audio │     │
│ └────────────────────────────────────┘     │
│ ┌────────────────────────────────────┐     │
│ │ Packet 3 (188 bytes)               │     │
│ │ └─ More video data                 │     │
│ └────────────────────────────────────┘     │
│ ... thousands more packets                 │
└────────────────────────────────────────────┘

```

5. So in terms of video streaming the flow from backend side of view is like :-


# Live Streaming Architecture Flow

## 🎯 Complete System Overview

```
┌─────────────┐                 ┌──────────────────────────────────────────┐                 ┌─────────────┐
│     OBS     │    RTMP         │         YOUR NODE.JS SERVER              │      HTTP       │   ANDROID   │
│  (Encoder)  │ ────────────>   │                                          │ <───────────── │     APP     │
│             │  Port: 1935     │  ┌────────────┐      ┌──────────────┐  │  Port: 8080     │ (ExoPlayer) │
└─────────────┘                 │  │    NMS     │      │   FFmpeg     │  │                 └─────────────┘
                                │  │ (Receiver) │ <──> │ (Converter)  │  │
                                │  └────────────┘      └──────────────┘  │
                                │                             │           │
                                │                             ↓           │
                                │                      ┌──────────────┐  │
                                │                      │  HLS Files   │  │
                                │                      │  .m3u8 + .ts │  │
                                │                      └──────────────┘  │
                                │                             ↑           │
                                │                      ┌──────────────┐  │
                                │                      │   Express    │  │
                                │                      │ (HTTP Server)│  │
                                │                      └──────────────┘  │
                                └──────────────────────────────────────────┘
```

---

##  Component Breakdown

### **1. OBS (Open Broadcaster Software)**
- **Role**: Video encoder and publisher
- **Protocol**: RTMP (Real-Time Messaging Protocol)
- **Output**: Live video stream
- **Connection**: `rtmp://your-server-ip:1935/live/teststream`

### **2. NMS (Node Media Server)**
- **Role**: RTMP receiver and broadcaster
- **Port**: 1935 (RTMP standard)
- **Function**: Relays stream to local consumers (FFmpeg)
- **Type**: Event-driven server

### **3. FFmpeg**
- **Role**: Video transcoder
- **Input**: RTMP stream from NMS (`rtmp://localhost:1935/live/teststream`)
- **Output**: HLS format (`.m3u8` + `.ts` chunks)
- **Process**: Separate OS process (not a thread)

### **4. Express Server**
- **Role**: HTTP file server
- **Port**: 8080
- **Function**: Serves HLS files to clients
- **Endpoint**: `http://your-server-ip:8080/stream/`

### **5. Android App (ExoPlayer)**
- **Role**: Video player
- **Protocol**: HTTP/HTTPS
- **Input**: HLS playlist (`index.m3u8`)
- **Function**: Downloads and plays video chunks

---

###  Server Initialization**

```
Time: 0ms
┌────────────────────────────────────────┐
│ 1. Node.js starts                      │
│ 2. NMS listens on port 1935 (RTMP)    │
│ 3. Express listens on port 8080 (HTTP)│
│ 4. Event handlers registered          │
│ 5. Waiting for connections...         │
└────────────────────────────────────────┘
```

---

###  OBS Connection**

```
Time: T+0s
┌─────────────┐
│     OBS     │
│  Streaming  │
└──────┬──────┘
       │ RTMP Handshake
       │ rtmp://server:1935/live/teststream
       ↓
┌──────────────┐
│     NMS      │ ──> Fires 'postPublish' event
│  Port: 1935  │
└──────────────┘
       │
       ↓
┌──────────────────────────────┐
│ Node.js Event Callback       │
│ - Detects stream started     │
│ - Launches FFmpeg process    │
└──────────────────────────────┘
```



###  FFmpeg Processing**

```
Time: T+1s
┌──────────────┐
│   FFmpeg     │
│   Process    │ ──────┐
└──────────────┘       │
       ↑               │
       │               │
       │ RTMP Client   │ File Writing
       │               │
       ↓               ↓
┌──────────────┐  ┌─────────────────────────┐
│     NMS      │  │   Filesystem            │
│  localhost   │  │   media/live/teststream/│
│  Port: 1935  │  │   ├── index.m3u8        │
└──────────────┘  │   ├── segment0.ts       │
                  │   ├── segment1.ts       │
                  │   ├── segment2.ts       │
                  │   ├── segment3.ts       │
                  │   └── segment4.ts       │
                  └─────────────────────────┘
```

**What FFmpeg Does:**
1. Connects to `rtmp://localhost:1935/live/teststream`
2. Decodes RTMP stream
3. Re-encodes to H.264 video + AAC audio
4. Segments into 4-second chunks (`.ts` files)
5. Generates/updates playlist (`index.m3u8`)
6. Deletes old chunks (keeps last 5)



###  HLS File Structure**

```
media/
└── live/
    └── teststream/
        ├── index.m3u8          ← Playlist (updated every 4 seconds)
        ├── segment0.ts         ← Video chunk 0 (4 seconds, ~500KB)
        ├── segment1.ts         ← Video chunk 1
        ├── segment2.ts         ← Video chunk 2
        ├── segment3.ts         ← Video chunk 3
        └── segment4.ts         ← Video chunk 4
```
---

##  Continuous Streaming Loop

```
┌─────────────────────────────────────────────────────────────┐
│                    CONTINUOUS LOOP                          │
│                                                             │
│  ┌────────────┐        ┌──────────┐        ┌─────────────┐ │
│  │    OBS     │ RTMP   │   NMS    │ RTMP   │   FFmpeg    │ │
│  │  Encoding  │───────>│ Relaying │<───────│  Converting │ │
│  └────────────┘        └──────────┘        └──────┬──────┘ │
│                                                    │        │
│                                                    │        │
│                                             Every 4 seconds │
│                                                    ↓        │
│                                            ┌────────────────┤
│                                            │ New chunk:     │
│                                            │ segment5.ts    │
│                                            │ Update:        │
│                                            │ index.m3u8     │
│                                            │ Delete:        │
│                                            │ segment0.ts    │
│                                            └────────┬───────┤
│                                                     │       │
│  ┌────────────┐                                    │       │
│  │  Android   │ HTTP GET /index.m3u8 (every 4s)    │       │
│  │    App     │<───────────────────────────────────┘       │
│  │  Polling   │                                            │
│  └────────────┘                                            │
└─────────────────────────────────────────────────────────────┘
```

**Timeline:**
- **T+0s**: segment0.ts created, playlist = [0]
- **T+4s**: segment1.ts created, playlist = [0,1]
- **T+8s**: segment2.ts created, playlist = [0,1,2]
- **T+12s**: segment3.ts created, playlist = [0,1,2,3]
- **T+16s**: segment4.ts created, playlist = [0,1,2,3,4]
- **T+20s**: segment5.ts created, segment0.ts **deleted**, playlist = [1,2,3,4,5]
- **T+24s**: segment6.ts created, segment1.ts **deleted**, playlist = [2,3,4,5,6]
- ...continues indefinitely

---

##  Threading & Process Model

```
┌───────────────────────────────────────────────────────────┐
│                   NODE.JS PROCESS (PID: 12345)            │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │           JavaScript Event Loop (Main Thread)       │ │
│  │                                                     │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐│ │
│  │  │   NMS   │  │ Express │  │  Event  │  │Callback││ │
│  │  │ Server  │  │ Server  │  │Handlers │  │ Queue  ││ │
│  │  └─────────┘  └─────────┘  └─────────┘  └────────┘│ │
│  │       ↑            ↑            ↑            ↑     │ │
│  └───────┼────────────┼────────────┼────────────┼─────┘ │
│          │            │            │            │       │
│  ┌───────┼────────────┼────────────┼────────────┼─────┐ │
│  │       │   libuv (C++ I/O Layer)│            │     │ │
│  │  ┌────▼─────┐  ┌──▼────┐  ┌───▼────┐  ┌────▼───┐ │ │
│  │  │  Thread  │  │Thread │  │ Thread │  │ Thread │ │ │
│  │  │  Pool 1  │  │Pool 2 │  │ Pool 3 │  │ Pool 4 │ │ │
│  │  └──────────┘  └───────┘  └────────┘  └────────┘ │ │
│  │   (Network)    (File I/O)   (Crypto)   (DNS)     │ │
│  └───────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────┘
                          ↕
                  IPC (Inter-Process Communication)
                          ↕
┌───────────────────────────────────────────────────────────┐
│                 FFMPEG PROCESS (PID: 67890)               │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐ │
│  │              FFmpeg's Own Thread Pool               │ │
│  │                                                     │ │
│  │  ┌──────────┐  ┌──────────┐  ┌────────┐  ┌──────┐ │ │
│  │  │  RTMP    │  │  Video   │  │ Audio  │  │ HLS  │ │ │
│  │  │  Client  │  │ Decoder  │  │Decoder │  │Muxer │ │ │
│  │  └──────────┘  └──────────┘  └────────┘  └──────┘ │ │
│  │       ↓             ↓             ↓          ↓     │ │
│  │  ┌──────────┐  ┌──────────┐  ┌────────┐  ┌──────┐ │ │
│  │  │  Demux   │  │  Video   │  │ Audio  │  │ File │ │ │
│  │  │          │  │ Encoder  │  │Encoder │  │ I/O  │ │ │
│  │  └──────────┘  └──────────┘  └────────┘  └──────┘ │ │
│  └─────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────┘
```

**Key Points:**
- **Node.js = Single-threaded JavaScript** (but multi-threaded I/O)
- **FFmpeg = Separate process** (multi-threaded internally)
- **They don't share memory** (communicate via RTMP and filesystem)

---

##  Port & Protocol Summary

| Component | Port | Protocol | Direction | Purpose |
|-----------|------|----------|-----------|---------|
| **OBS** | - | RTMP | Outbound | Publish stream |
| **NMS** | 1935 | RTMP | Inbound | Receive stream |
| **FFmpeg** | - | RTMP | Outbound (to localhost) | Read stream |
| **Express** | 8080 | HTTP | Inbound | Serve files |
| **Android** | - | HTTP | Outbound | Fetch playlist & chunks |

---

##  Network Flow

```
Internet
   │
   ├──────────────────────────────────────────┐
   │                                          │
   ↓                                          ↓
┌─────────┐                            ┌──────────┐
│   OBS   │                            │ Android  │
│ Studio  │                            │   App    │
└────┬────┘                            └─────┬────┘
     │                                       │
     │ RTMP (WAN)                           │ HTTP (WAN)
     │ Port: 1935                           │ Port: 8080
     │                                       │
     └──────────┬────────────────────────────┘
                │
        ┌───────▼────────┐
        │  YOUR SERVER   │
        │  (Cloud/Local) │
        └───────┬────────┘
                │
        ┌───────┴─────────┐
        │                 │
        ↓                 ↓
   ┌─────────┐      ┌──────────┐
   │   NMS   │ RTMP │  Express │
   │Port:1935│◄─────┤Port: 8080│
   └────┬────┘      └────┬─────┘
        │ localhost      │
        │ (loopback)     │
        ↓                ↓
   ┌─────────┐      ┌──────────┐
   │ FFmpeg  │─────►│HLS Files │
   │ Process │ write│.m3u8/.ts │
   └─────────┘      └──────────┘
```

---

##  Event Timeline

```
┌──────────┬─────────────────────────────────────────────────────┐
│   Time   │                     Event                           │
├──────────┼─────────────────────────────────────────────────────┤
│   0ms    │ Node.js starts                                      │
│  10ms    │ NMS listening on :1935                              │
│  15ms    │ Express listening on :8080                          │
│  20ms    │ Server ready, waiting...                            │
│          │                                                     │
│ 5000ms   │ OBS connects via RTMP                               │
│ 5001ms   │ 'postPublish' event fires                           │
│ 5002ms   │ FFmpeg process spawns                               │
│ 5500ms   │ FFmpeg connects to NMS (localhost)                  │
│ 6000ms   │ FFmpeg outputs first chunk (segment0.ts)            │
│ 6001ms   │ index.m3u8 created                                  │
│          │                                                     │
│ 10000ms  │ Android app requests index.m3u8                     │
│ 10100ms  │ Express serves index.m3u8                           │
│ 10200ms  │ Android requests segment0.ts                        │
│ 10300ms  │ Express serves segment0.ts                          │
│ 10400ms  │ ExoPlayer starts playback                           │
│          │                                                     │
│ 14000ms  │ Android polls index.m3u8 again                      │
│ 14100ms  │ Detects segment1.ts available                       │
│ 14200ms  │ Downloads segment1.ts                               │
│ 14300ms  │ Seamless playback continues...                      │
│          │                                                     │
│ 30000ms  │ User stops OBS stream                               │
│ 30001ms  │ 'donePublish' event fires                           │
│ 30002ms  │ FFmpeg process killed (SIGINT)                      │
│ 30100ms  │ HLS files remain on disk                            │
│ 30500ms  │ Android continues playing buffered chunks           │
│ 35000ms  │ Playback ends (no new chunks)                       │
└──────────┴─────────────────────────────────────────────────────┘
```



### **Data Journey:**
1. **OBS** encodes camera/screen → RTMP packets
2. **NMS** receives RTMP → broadcasts to localhost
3. **FFmpeg** reads RTMP → converts to HLS → saves to disk
4. **Express** serves files → sends via HTTP
5. **Android** downloads chunks → plays seamlessly

### **Key Technologies:**
- **RTMP**: Low-latency streaming protocol
- **HLS**: Adaptive streaming format (works everywhere)
- **FFmpeg**: Swiss Army knife of video processing
- **Node.js**: Async event-driven server
- **Express**: Flexible HTTP server


