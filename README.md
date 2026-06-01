# Local Media Player Deployment Guide (Offline Pod)

## Overview

This document provides the technical instructions required to convert a secondary iOS device into a secure, offline local media player. The architecture eliminates cloud dependencies, stops data telemetry, and reduces the network attack surface of the asset.

---

## Objective

The purpose of this guide is to configure a secondary iOS device as a dedicated, standalone local media player. By applying strict system hardening, total telemetry suppression, and network segmentation, the configuration eliminates data transmissions to external cloud services and minimizes both the physical and digital attack surface of the asset.

---

## Environment & Requirements

-   Host System: Windows 10 or Windows 11
    
-   Target Device: iPhone XR (or similar iOS asset)
    
-   Network: Local Wi-Fi network (isolated VLAN recommended)
    
---

## Technical Implementation

### 1. Host Software Installation

Open PowerShell as Administrator and execute the following commands to install the audio extraction binaries via the Windows Package Manager:

```powershell
winget install yt-dlp
winget install gyan.ffmpeg
```

Close and restart the PowerShell window to initialize the new environment path variables.

---

### 2. Audio Acquisition

Create a target directory on your Windows filesystem for the media storage. Use the terminal to navigate to this folder and run the execution command:

```powershell
yt-dlp -x --audio-format mp3 --audio-quality 0 "TARGET_YOUTUBE_URL"
```

> [![asciicast](https://asciinema.org/a/1171726.svg)](https://asciinema.org/a/1171726)

To automate the download of multiple tracks, replace the individual video URL with a public or unlisted YouTube playlist URL. The binary will process the queue sequentially.

---

### 3. Metadata and Artwork Injection

Files extracted from streaming platforms lack structured ID3 tags. You must write this data directly into the file headers before network deployment.

1.  Download and install MusicBrainz Picard.
    
2.  Navigate to Options > Cover Art. Select the checkbox for **Embed cover images into tags**.
    
3.  Drag the downloaded audio directory into the left pane (Unclustered Files).
    
4.  Highlight the tracks and click **Scan** on the top menu bar. This triggers acoustic fingerprinting to identify the audio waves against the central database.
    
5.  Review the matches in the right pane. Click the album to verify the cover art preview in the bottom right corner. Select the matched albums and click **Save**.
    
6.  **For Failed Tracks (Left Pane):** If a file fails matching and remains under "Unclustered Files," highlight it and click **Cluster** then **Lookup** to force a text-based search. 

7.  **For Manual Fixes:** If it still fails, right-click the empty CD icon in the bottom right corner of Picard, choose **Change Front Cover**, select a manually downloaded JPEG/PNG image, and click **Save** directly from the left pane (files do not need to move to the right pane to be saved).

---

### 4. Wireless Target Deployment

The transfer occurs entirely over the local network layer, bypassing Apple ecosystem software.

1.  Download VLC for Mobile from the iOS App Store.
    
2.  Sign out of the Apple ID on the device immediately after installation.
    
3.  Open VLC, select the Network tab, and enable the **Sharing via WiFi** toggle. Note the local IP address displayed.
    
4.  Ensure the Windows PC is connected to the same network segment as the iOS device. Open a web browser on the PC and input the target IP address into the address bar.
    
5.  Drag the tagged audio files from your Windows directory into the browser window.
    
6.  Turn off the Sharing via WiFi toggle in VLC as soon as the transfer status hits 100 percent.
    
---

## Asset Hardening & Network Isolation

Execute these configuration modifications on the iOS target device to restrict telemetry and secure the perimeter.

|**Setting Directory**|**Action Required**|**Purpose**|
|-------------------|-------------------|-----------|
|**Lockdown Mode**|Settings > Privacy & Security > Lockdown Mode > Turn On|Strips complex web binaries, blocks unauthorized wired access, and stops unauthorized configuration profiles.|
|**Primary Telemetry**|Settings > Privacy & Security > Analytics & Improvements|Toggle off all options including Share iPhone Analytics, Share iCloud Analytics, and Improve Siri & Dictation to halt diagnostic data transmissions.|
|**Buried Telemetry**|Settings > Privacy & Security > Location Services > System Services|Scroll to the bottom interface section labeled Product Improvement. Deactivate iPhone Analytics, Routing & Traffic, and Improve Maps.|
|**Ad Identifiers**|Settings > Privacy & Security > Apple Advertising|Turn off Personalized Ads.|
|**App Token Tracking**|Settings > Privacy & Security > Tracking|Deactivate Allow Apps to Request to Track to completely block cross-application tracking requests.|
|**Voice Processing**|Settings > Siri & Search (or Apple Intelligence & Siri)|Turn off Listen for "Hey Siri", Press Side Button for Siri, and Show when Locked. Select Turn Off Siri when prompted to disable microphone capture and vocal parsing.|
|**Lock Screen Control**|Settings > Face ID & Passcode > Allow Access When Locked|Toggle off Control Center, Siri, Reply with Message, and Accessories to block physical interface manipulation while the device is locked.|
|**Radio Protocols**|Disable Bluetooth, Location Services, and set AirDrop to Receiving Off|Eliminates localized wireless attack vectors and proximity tracking.|
|**Access Control**|Set complex alphanumeric passcode|Prevents physical brute-force access.|
|**Storage Security**|Enable Erase Data (Bottom of Face ID & Passcode menu)|Triggers cryptographic erasure of the file system keys after 10 failed passcode attempts.|
|**Network Layer**|Forget primary Wi-Fi network; isolate on a Guest VLAN or maintain Airplane Mode|Prevents outbound device telemetry and stops lateral movement on the home network.|

---

## Verification

Open the Audio tab in VLC on the iOS device. Swipe down to trigger a local file system re-index. The media library must display correct artist classification, album structuring, and embedded cover art without active external network connections.
