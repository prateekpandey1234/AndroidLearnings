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

5. 
