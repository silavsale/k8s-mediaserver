kubectl apply -f media-stack.yaml

kubectl get deployments

sudo chmod -R 777 /srv/media/Sonarr/tvshows

kubectl delete -f media-stack.yaml

- Jellyfin: serverIP:8096
- Sonarr: serverIP:8989
- Radarr: serverIP:7878
- Prowlarr: serverIP:9696
- Lidarr: serverIP:8686
- Readarr: serverIP:8787
- Homarr: serverIP:7575
- qBittorrent: serverIP:8080

### How the Workflow Works

# Prowlarr (Indexer Manager):

What it does: Prowlarr connects to multiple torrent indexers (or usenet indexers if you prefer) and aggregates search results.
Role: It supplies Radarr with the information it needs to find available movie torrents.
Radarr (Movie Manager):

What it does: Radarr lets you add movies to your “wanted” list. It regularly searches for the movies you’ve added by using indexers (via Prowlarr) and then automatically sends the download request to your chosen download client.
Role: It automates movie searches, download requests, and post-download handling (like renaming and moving files).
qBittorrent (Download Client):

What it does: qBittorrent receives the download requests (sent by Radarr) and handles the torrent downloading process.
Role: It downloads the movie files to a designated location on your server.
Jellyfin (Media Server):

What it does: Jellyfin scans the folder where the movies are stored and makes them available in its library for streaming.
Role: It serves the movies to any device on your local network.
Step-by-Step Setup
Step 1: Configure Prowlarr
Access Prowlarr:
Open your browser and navigate to:
http://<your-server-ip>:9696

Add Indexers:

Go to the Indexers section in Prowlarr.
Add and configure your preferred torrent (or usenet) indexers.
Tip: Make sure you choose indexers that work in your region and that you are comfortable using.
Test Indexers:
Ensure that the indexers return results when you search from within Prowlarr.

Step 2: Configure Radarr
Access Radarr:
Open your browser and navigate to:
http://<your-server-ip>:7878

Set Up Download Client (qBittorrent):

Go to Settings > Download Clients.
Add a new download client, select qBittorrent.
Enter the details:
Host: Use your server’s IP or localhost (if running with host networking)
Port: Typically 8080 (if that’s what you set up in your qBittorrent configuration)
Authentication: Provide the username and password as configured in your qBittorrent instance.
Test the connection to ensure Radarr can talk to qBittorrent.
Add Prowlarr as an Indexer (if needed):

In Radarr’s Settings > Indexers, you may be able to add Prowlarr if it offers a direct integration.
Alternatively, Prowlarr can supply indexers for both movies and TV (if using Sonarr for TV) so that Radarr automatically uses them.
Set the Movie Folder:

In Radarr’s settings, configure the Root Folder for movies. For example, set it to the folder that corresponds to your hostPath mount for movies (e.g., /data/movies inside the container, which is mapped to /srv/media/Radarr/movies on your host).
Add Movies:

Use the Add Movie button to add movies to your library.
Radarr will then automatically search for available torrents using the indexers (via Prowlarr) and, if it finds a match, send the download command to qBittorrent.
Step 3: Configure qBittorrent
Access qBittorrent:
Open your browser and navigate to:
http://<your-server-ip>:8080

Configure Download Folders:

In qBittorrent’s settings, ensure that the default download folder is set appropriately (often this is set to a “Downloads” folder).
Radarr usually expects to have completed downloads moved to the Movies folder. In Radarr, enable Completed Download Handling so that after the torrent finishes, it will move the movie file from the download folder to your configured movie folder.
Ensure Port Accessibility:

Double-check that both TCP and UDP ports for torrenting (e.g., 6881) are allowed through your firewall if needed.
Step 4: Configure Jellyfin
Access Jellyfin:
Open your browser and navigate to:
http://<your-server-ip>:8096

Add Your Movie Library:

Go to the Dashboard and then Libraries.
Click + Add Library.
For a Movies library, set the folder path to the container’s mount point (for example, /data/Movies).
This is mapped to /srv/media/Radarr/movies on your host.
Complete the library setup and perform a scan so that Jellyfin picks up the new movie files.
Optional – Set Up Automatic Library Scans:
You can configure Jellyfin to scan periodically or trigger a scan manually whenever Radarr completes a download.

Step 5: Test the Workflow
Add a New Movie in Radarr:

Pick a movie and add it to Radarr.
Radarr will check with Prowlarr for available torrents.
If found, Radarr sends the download request to qBittorrent.
Monitor qBittorrent:

Watch qBittorrent’s interface to see the torrent start downloading.
Once the download completes, Radarr should automatically import the file into your movies folder.
Check in Jellyfin:

After Radarr imports the movie, trigger a library scan in Jellyfin.
The new movie should appear in your Jellyfin Movies library and be ready for streaming.
