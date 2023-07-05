<table
  id="player"
  style="
    background-color: #1b1b1c;
    color: #fff;
    font-family: Helvetica, Arial;
    font-size: 6vmin;
    border-radius: 20px;
    filter: opacity(0.8);
    width: 100%;
    height: 100%;
	box-shadow: 0 10px 20px -10px rgba(0,0,0,.75);
  "
>
  <tr height="100%">
    <td width="20%" align="center">
      <img
        id="player-album-art"
        style="margin: 1em; max-height: 50vmin; border-radius: 10px"
      />
    </td>
    <td width="80%">
      <div style="margin: 1em 1em 1em 0">
        <div id="player-song" style="font-size: 2em"></div>
        <div
          id="player-artist"
          style="font-size: 1.3em; margin-bottom: 1em"
        ></div>
        <a
          id="player-song-link"
          href="https://open.spotify.com/user/31437s53qjwmftue3bzlzg4zptpe?si=f66d1651bad348c2"
          target="_blank"
          style="
            color: #fff;
            text-decoration: none;
            padding: 0.2em;
            margin-right: 0.2em;
          "
        >
          <img src="https://i.hizliresim.com/pkgtvs9.png" id="player-song-link">
        </a>
        <a
          id="player-artist-link"
          href=""
          target="_blank"
          style="
            color: #fff;
            text-decoration: none;
          "
        >
          </a
        >
        <a
          id="player-album-link"
          href=""
          target="_blank"
          style="
            color: #fff;
            text-decoration: none;
          "
        >
          </a
        >
        <a
          id="player-mp3-link"
          style="
            color: #fff;
            text-decoration: none;
          "
        >
        </a
        >
        <div id="player-status" style="margin: 2em 0 1em 0"></div>
        <div
          id="player-time"
          style="position: relative; float: right; top: -2em; font-weight: bold"
        ></div>
        <div
          id="player-progress-back"
          style="border: 0.15em solid #eee; height: 1em; border-radius: 10px"
        >
          <div
            id="player-progress"
            style="
              background-color: #eee;
              border: 0.1em solid transparent;
              height: 0.75em;
              border-radius: 10px;
              transition: width 0.2s;
            "
          ></div>
        </div>
      </div>
    </td>
  </tr>
  <div
    id="player-background"
    class="background"
    style="
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      z-index: -10;
      background-position: center center;
      background-size: 100%;
      filter: blur(0.4em);
      position: absolute;
	  border-radius: 20px;
      transition: background-image 0.5s ease-in;
    "
  ></div>
</table>

<script>
  var serviceHost = "https://spotify.lykiaofficial.workers.dev";
var spotifyUser = "lyki";

var songData, progressSeconds, totalSeconds, progressInterval, updateInterval;

function updatePlayer() {
  fetch(`${serviceHost}/get-now-playing`)
    .then((response) => response.json())
    .then((data) => {
      if (data.hasOwnProperty("ERROR")) {
        document.getElementById(
          "player-song"
        ).innerHTML = `${spotifyUser} hiçbir şey dinlemiyor.`;
        document.getElementById("player-artist").innerHTML = "  ";
        return;
      }
      songData = data;
      document.getElementById("player-song").innerHTML = data.item.name;
      document.getElementById("player-artist").innerHTML =
        data.item.artists[0].name;
      document.getElementById("player-status").innerHTML = data.is_playing
        ? `▶ ${spotifyUser} dinliyor...`
        : `▶ ${spotifyUser} durdurdu...`;
      document
        .getElementById("player-album-art")
        .setAttribute("src", data.item.album.images[1].url);
      document
        .getElementById("player-progress")
        .setAttribute(
          "style",
          document.getElementById("player-progress").getAttribute("style") +
            `width: ${(data.progress_ms * 100) / data.item.duration_ms}%`
        );

      document.getElementById(
        "player-background"
      ).style.backgroundImage = `url(${data.item.album.images[1].url})`;

      // Set the links to spotify stuff
      document
        .getElementById("player-song-link")
        .setAttribute("href", data.item.external_urls.spotify);
      document
        .getElementById("player-artist-link")
        .setAttribute("href", data.item.artists[0].external_urls.spotify);
      document
        .getElementById("player-album-link")
        .setAttribute("href", data.item.album.external_urls.spotify);
      document
        .getElementById("player-mp3-link")
        .setAttribute("href", data.item.preview_url);

      // Hide mp3 link if the song does not provide that
      if (data.item.preview_url == null) {
        document
          .getElementById("player-mp3-link")
          .setAttribute("style", "display: none;");
      }

      // Timer to show updates on progress bar and time
      // https://stackoverflow.com/questions/5517597/plain-count-up-timer-in-javascript
      progressSeconds = Math.ceil(songData.progress_ms / 1000);
      totalSeconds = Math.ceil(songData.item.duration_ms / 1000);
      
      // Clear previous progress interval
      clearInterval(progressInterval);
      
      // Process progress only if a song is in 'playing' state
      if (songData.is_playing) {
        progressInterval = setInterval(setProgress, 1000);
      } else {
        setProgress();
      }

      // Hide all the extra things in mobile (<410px)
      if (document.getElementById("player").clientWidth < 410) {
        // Hide links
        document.getElementById("player-song-link").style.display = "none";
        document.getElementById("player-artist-link").style.display = "none";
        document.getElementById("player-album-link").style.display = "none";
        document.getElementById("player-mp3-link").style.display = "none";

        // Hide duration
        document.getElementById("player-time").style.display = "none";
      }
    });
}

function setProgress() {
  if (progressSeconds > totalSeconds) {
    clearInterval(progressInterval);
    updatePlayer(); // Yeni şarkıya geçmek için updatePlayer() fonksiyonunu çağır
    progressInterval = setInterval(setProgress, 1000); // 5 saniyede bir updatePlayer() fonksiyonunu çağırmak için progressInterval'ı yeniden başlat
    return;
  }
  ++progressSeconds;
  var totalLabel =
    pad(parseInt(totalSeconds / 60)) + ":" + pad(totalSeconds % 60);
  var progressLabel =
    pad(parseInt(progressSeconds / 60)) + ":" + pad(progressSeconds % 60);
  document.getElementById("player-time").innerHTML =
    progressLabel + " / " + totalLabel;
  document.getElementById("player-progress").style.width = `${
    (progressSeconds * 100) / totalSeconds
  }%`;
}

function pad(val) {
  var valString = val + "";
  if (valString.length < 2) {
    return "0" + valString;
  } else {
    return valString;
  }
}

// Load player for the first time
updatePlayer();

// Update player every 5 seconds
updateInterval = setInterval(updatePlayer, 5000);
</script>
