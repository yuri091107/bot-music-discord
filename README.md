
```bash
pip install discord.py yt-dlp spotipy

Step 2: Install FFmpeg
FFmpeg is a powerful multimedia framework used to handle audio and video files. Follow these steps to install FFmpeg:

For Windows:
Download FFmpeg:

Go to the FFmpeg download page.
Select the appropriate version for Windows and download the zip file.
Extract FFmpeg:

Extract the contents of the zip file to a folder, e.g., C:\ffmpeg.
Add FFmpeg to PATH:

Open Control Panel.
Go to System and Security > System.
Click on Advanced system settings.
Click on Environment Variables.
Under System variables, find the PATH variable, select it, and click Edit.
Click New and add the path to the FFmpeg bin directory (e.g., C:\ffmpeg\bin).
Click OK to close all dialog boxes.
Verify Installation:

Open a new command prompt and type ffmpeg -version. If installed correctly, FFmpeg version information will be displayed.
For macOS:
Install Homebrew (if not already installed):

Open the Terminal app and paste the following command:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

Install FFmpeg:

Run the following command in the Terminal:
brew install ffmpeg

Install FFmpeg:

Open your terminal and run the following command based on your distribution:
sudo apt update
sudo apt install ffmpeg

Verify Installation:

Type ffmpeg -version in the terminal. You should see FFmpeg version information if the installation was successful.
Bot Commands
!join: Join the voice channel of the user.
!leave: Leave the current voice channel.
!play <URL or search query>: Play a song from YouTube or Spotify.
!pause: Pause the current song.
!resume: Resume the paused song.
!stop: Stop the current song.
!skip: Skip to the next song in the queue.
!swap <YouTube URL or Spotify URL or song number>: Swap the current song with a specific song.
Running the Bot
Replace 'YOUR_BOT_TOKEN', 'YOUR_SPOTIPY_CLIENT_ID', and 'YOUR_SPOTIPY_CLIENT_SECRET' with your actual bot token and Spotify API credentials in the code. Then, run the bot with:
bot.run('YOUR_BOT_TOKEN')

