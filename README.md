# YT-Downloader
Fast, convenient and automated YT-Downloader that allows both audio and video. Windows/CLI
Execute ffmpeg first and then the downloader.

DISCLAIMER!
⚠️ Windows may warn about "Unknown Publisher" because this app isn’t code-signed. This is expected for open-source projects without a commercial certificate. The code is open source and available in this repository, so you can review
or build it yourself if you prefer.

## Licenses
This software uses code of [FFmpeg](http://ffmpeg.org) licensed under the 
[LGPLv2.1 or later](http://www.gnu.org/licenses/old-licenses/lgpl-2.1.html).  
The FFmpeg source code can be downloaded [here](https://ffmpeg.org/download.html).

## Python Script
```python
# youtube.py
import sys
from pathlib import Path
from pytubefix import YouTube
from pytubefix.cli import on_progress
import subprocess
import os

def get_quality_options(streams, is_audio=False):
    qualities = []
    seen = set()
    for s in streams:
        q = s.abr if is_audio else s.resolution
        if q and q not in seen:
            qualities.append(q)
            seen.add(q)
    return qualities

def select_quality(streams, quality, is_audio=False):
    if is_audio:
        return streams.filter(abr=quality).first()
    else:
        return streams.filter(res=quality).first()

def main_menu():
    os.system('cls' if os.name == 'nt' else 'clear')
    print("YouTube Downloader Menu")
    print("1. Download Music (audio only)")
    print("2. Download Video (video + audio)")
    print("3. Exit")
    choice = input("Select an option (1/2/3): ").strip()
    return choice

def ask_url():
    if len(sys.argv) > 1:
        return sys.argv[1]
    return input('Enter Youtube URL: ').strip()

def download_music(yt):
    audio_streams = yt.streams.filter(only_audio=True, file_extension='mp4').order_by('abr').desc()
    qualities = get_quality_options(audio_streams, is_audio=True)
    if not qualities:
        print("No audio streams found.")
        return
    print("Available audio qualities:")
    for idx, q in enumerate(qualities, 1):
        print(f"{idx}. {q}")
    while True:
        try:
            sel = int(input("Select quality: ")) - 1
            if 0 <= sel < len(qualities):
                break
            else:
                print("Invalid selection. Please try again.")
        except ValueError:
            print("Invalid input. Please enter a number.")
    chosen_quality = qualities[sel]
    stream = select_quality(audio_streams, chosen_quality, is_audio=True)
    audio_library = Path.home() / "Music"
    audio_library.mkdir(parents=True, exist_ok=True)
    audio_path = audio_library / f'{yt.title}_audio_{chosen_quality}.mp4'
    print(f'Downloading Audio ({chosen_quality})...')
    stream.download(output_path=audio_library, filename=audio_path.name)
    print(f'Audio saved to: {audio_path}')

def download_video(yt):
    video_streams = yt.streams.filter(progressive=True, file_extension='mp4').order_by('resolution').desc()
    if not video_streams:
        # fallback to adaptive (highest quality, but needs merging)
        print("Available video qualities:")
        for idx, q in enumerate(video_qualities, 1):
            print(f"{idx}. {q}")
        while True:
            try:
                vsel = int(input("Select video quality: ")) - 1
                if 0 <= vsel < len(video_qualities):
                    break
                else:
                    print("Invalid selection. Please try again.")
            except ValueError:
                print("Invalid input. Please enter a number.")
        chosen_vq = video_qualities[vsel]
        vstream = select_quality(video_streams, chosen_vq)
        print("Available audio qualities:")
        for idx, q in enumerate(audio_qualities, 1):
            print(f"{idx}. {q}")
        while True:
            try:
                asel = int(input("Select audio quality: ")) - 1
                if 0 <= asel < len(audio_qualities):
                    break
                else:
                    print("Invalid selection. Please try again.")
            except ValueError:
                print("Invalid input. Please enter a number.")
        print("Available video qualities:")
        for idx, q in enumerate(qualities, 1):
            print(f"{idx}. {q}")
        while True:
            try:
                sel = int(input("Select quality: ")) - 1
                if 0 <= sel < len(qualities):
                    break
                else:
                    print("Invalid selection. Please try again.")
            except ValueError:
                print("Invalid input. Please enter a number.")
        chosen_quality = qualities[sel]
        stream = select_quality(video_streams, chosen_quality)
        for idx, q in enumerate(audio_qualities, 1):
            print(f"{idx}. {q}")
        asel = int(input("Select audio quality: ")) - 1
        chosen_aq = audio_qualities[asel]
        astream = select_quality(audio_streams, chosen_aq, is_audio=True)
        video_library = Path.home() / "Videos"
        video_library.mkdir(parents=True, exist_ok=True)
        video_path = video_library / f'{yt.title}_video_{chosen_vq}.mp4'
        audio_path = video_library / f'{yt.title}_audio_{chosen_aq}.mp4'
        print(f'Downloading Video ({chosen_vq})...')
        vstream.download(output_path=video_library, filename=video_path.name)
        print(f'Video saved to: {video_path}')
        print(f'Downloading Audio ({chosen_aq})...')
        astream.download(output_path=video_library, filename=audio_path.name)
        print(f'Audio saved to: {audio_path}')
        print("\nNote: For highest quality, you'll need to merge video and audio files using a tool like ffmpeg.")
    else:
        qualities = get_quality_options(video_streams)
        print("Available video qualities:")
        for idx, q in enumerate(qualities, 1):
            print(f"{idx}. {q}")
        sel = int(input("Select quality: ")) - 1
        chosen_quality = qualities[sel]
        stream = select_quality(video_streams, chosen_quality)
        video_library = Path.home() / "Videos"
        video_library.mkdir(parents=True, exist_ok=True)
        video_path = video_library / f'{yt.title}_video_{chosen_quality}.mp4'
        print(f'Downloading Video ({chosen_quality})...')
        stream.download(output_path=video_library, filename=video_path.name)
        print(f'Video saved to: {video_path}')

if __name__ == "__main__":
    
    while True:
        choice = main_menu()
        if choice == "1":
            yt_url = ask_url()
            yt = YouTube(yt_url, on_progress_callback=on_progress)
            download_music(yt)
        elif choice == "2":
            yt_url = ask_url()
            yt = YouTube(yt_url, on_progress_callback=on_progress)
            download_video(yt)
        elif choice == "3":
            print("Exiting...")
            break
        else:
            print("Invalid option. Please try again.")
