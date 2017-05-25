# Binding for Spotify and Spotify Connect Devices

This binding implements a bridge to the Spotify Player Web API and makes it possible to discover Spotify Connect Devices on your network using you Spotify Premium account.
## Configuring the binding
The binding requires you to register an Application with Spotify Web API at https://developer.spotify.com - this will get you a set of Client ID and Client Secret parameters to be used by your binding configuration.

###Create Spotify Application
Follow the instructions in the tutorial at https://developer.spotify.com/web-api/tutorial/. Skip into and follow instructions under:

 1. Setting Up Your Account
 1. Registering Your Application
 
- Step 6: entering Website information can be skipped.
- Step 7: setting Redirect URIs is **very important**

When registering your new Spotify Application for OpenHAB Spotify Bridge you have to specify the allowed "Redirect URIs" aka white-listed addresses. Here you have to specify the URL to the Bridge Authentication Servlet on your server. 

If you run your OpenHAB server on "http://openhab.mydomain.com:8080"  you should add "http://openhab.mydomain.com:8080/connectspotify/authorize" to the Redirect URIs. 

This is important since the authentication process with Spotify takes place using your client web browser. Once you have authenticated with Spotify, this Redirect URI is where authorization tokens for your OpenHAB Spotify Brigde will be sent and they have to be received by the servlet on "/connectspotify".

## Configure binding

1. Install bind and make sure the Spotify Binding is listen on your server
1. Complete the Spotify Application Registation if you have not already done so, see above.
1. Make sure you have your Spotify Application _Client ID_ and _Client Secret_ identities available.
1. Goto to your preferred OpenHAB admin UI and add a new Thing. Select the **"Spotify Player Bridge"**. Choose new Id for the player, unless you like the generated one, put in the _Client ID_ and _Client Secret_ from the Spotify Application registration in their respective fields of the bridge configuration. You can leave the refreshPeriod and refreshToken as is. Save the bridge.
1. The bridge thing will stay in state _INITIALIZING_ and eventually go OFFLINE - this is fine. You have to authenticate this bridge with Spotify.
1. Go to the simple authentication page of your server: "http://openhab.mydomain.com:8080/connectspotify/". Your newly added bridge should be listed there.
1. Press the _"Authenticate Player with Spotify WebAPI"_ button. This will take you either to the login page of Spotify or directly to the authorization screen. Login and/or authorize the application. If the Redirect URIs are correct you will be returned to a results page with all technical identifiers of the authentication process. If not, go back to your Spotify Application and ensure you have the right Redirect URIs.
1. The binding will be updated with a _refreshToken_ and go ONLINE. The _refreshToken_ is used to re-authenticate the bridge with Spotify Connect WebAPI whenever required. An authentication token is valid for an hour so there is re-authentication thread running to refresh the internal _accessToken_.

Now that you have got your bridge ONLINE it is time to discover your devices! Go to OpenHAB Inbox and search for Spotify Connect Devices. Any device active should show up immediately. 

If no devices show up you can start Spotify App on your PC/Mac/iOS/Android and start playing on your devices as you run discovery. This should make both devices and Spotify Apps discoverable.  You may have to trigger the OpenHAB discovery serveral times as bridge will only find active devices known by the Spotify WebAPI at the time the discovery is triggered.

Should the bridge configuration be broken for any reason, the authentication procedure can be reinitiated from step 6 whenever required. You can force reinitialization simply by removing the refreshToken of the bridge configuration and save. Make sure you leave _Client ID_ and _Client Secret_ intact and correct. Press save.

## Supported Things

All Spotify Connect capable devices should be discoverable through this binding. If you can control them from Spotify Player app on your PC/Mac/iPhone/Android/xxx you should be able to add it as a thing.

## Discovery

As long as Spotify Connect devices are available on the network of your server they should show up whenever you initiate discovery of things. If it is not working, try to connect your computer to the same network as the OpenHAB server and make sure Spotify App can find the devices from your computer.

## Channels

### Bridge / Player
The channels on the bridge are the ones used to both control the active device and get details of currently playing music.

__Common Channels:__
| Channel Type ID       | Item Type    | Description  |
|-----------------------|------------------------|--------------|
| deviceName 		| String 	| Name of the currently active Connect Device |
| deviceVolume 		| Dimmer 	| Get or set the active Connect Device volume |
| trackPlay 		| String 	| Set which music  to play on the active device. This channel accepts Spotify URIs and Hrefs. |
| trackPlayer 		| Player 	| The Player Control of the active device. Play/Pause/Next/Previous commands. |
| trackRepeat 		| String 	| The current device Repeat Mode setting |
| trackShuffle 		| Switch 	| The current device  Shuffle setting |
| trackName 		| String 	| The name of the currently played track |
| trackDuration 	| String 	| The duration (m:ss) of the currently played track|
| trackProgress 	| String 	| The Progress (m:ss) of the currently played track. This is updated as the refreshPeriod of the Bridge.|
| albumName 		| String 	| Album Name of the currently played track|
| artistName 		| String 	| Artist Name of the currently played track | 

__Advanced Channels:__
| Channel Type ID       | Item Type    | Description  |
|-----------------------|------------------------|--------------|
| accessToken			| String	| The current accessToken used in communication with WebAPI. This can be used in client-side scripting towards the WebAPI if you would like to maintain your playlists etc. |
| currentlyPlayedTrackId | String | Track Id of the currently played track. |
| currentlyPlayedTrackHref | String | Track Href of the currently played track. |
| currentlyPlayedTrackUri | String | Track Uri of the currently played track. |
| currentlyPlayedTrackType | String | Type of the currently played track. |
| currentlyPlayedTrackNumber | String | Number of the track on the album/record. |
| currentlyPlayedTrackDiscNumber | String | Disc Number of the track on the album/record. |
| currentlyPlayedTrackPopularity | Dimmer | Currently played track popularity |
| currentlyPlayedAlbumId | String | Album Id of the currently played track. |
| currentlyPlayedAlbumHref | String | Album Href of the currently played track. |
| currentlyPlayedAlbumUri | String | Album Uri of the currently played track. |
| currentlyPlayedAlbumType | String | Album Type of the currently played track. |
| currentlyPlayedArtistId | String | Artist Id of the currently played track. |
| currentlyPlayedArtistHref | String | Artist Href of the currently played track. |
| currentlyPlayedArtistUri | String | Artist Uri of the currently played track. |
| currentlyPlayedArtistType | String | Artist Type of the currently played track. |

### Devices 
There will be channels on the bridge and on the devices. The functionality of the bridge channels will be overlapping some of the ones of the devices. 

__Common Channels:__
| Channel Type ID       | Item Type    		| Description  |
|-----------------------|-----------------------|--------------|
| trackPlay 		| String 		| Track to play on the device. Assigning a track, playlist, artist etc will activate the device and make it the currently playing device. |
| deviceName 		| String 		| Name of the device |
| deviceVolume 		| Dimmer 		| Volume setting for the device |
| devicePlayer 		| Player 		| Player Control of the device |
| deviceShuffle 	| Switch 		| Turn on/off shuffle play |

__Advanced Channels:__
| Channel Type ID       | Item Type    		| Description  |
|-----------------------|-----------------------|--------------|
| deviceId | String | The Spotify Connect device Id. |
| deviceType | String | The type of device e.g. Speaker, Smartphone |
| deviceActive | Switch | Indicated if the device is active or not. Should be the same as Thing status ONLINE/OFFLINE |
| deviceRestricted | Switch | Indicates if this device allows to be controlled by the API or not. If restricted it cannot be controlled |


## Full Example

This is a roughly what I have used to test the binding in development. Auto created items, and I added separate items in files for the Player channel. Don't know how to set it up in sitemap under basicui. So I mapped a Switch to the Player channel and referenced that from the sitemap to start/stop playing. The channel supports step tracks forward/backward.

spotify.items:

    Switch device1Play  {channel="spotify:device:xxx:3b4...ed4:devicePlay"}
    Switch device2Play  {channel="spotify:device:xxx:abc...123:devicePlay"}

spotify.sitemap

    sitemap spotify label="Spotify Sitemap" {
         
        Frame label="Spotify Player Info" {
            Text item=spotify_player_xxx_trackRepeat label="Currently Player repeat mode: [%s]"
            Text item=spotify_player_xxx_trackShuffle label="Currently Player shuffle mode: [%s]"
            Text item=spotify_player_xxx_trackProgress label="Currently Played track progress: [%s]"
            Text item=spotify_player_xxx_trackDuration label="Currently Played track duration: [%s]"
            Text item=spotify_player_xxx_trackName label="Currently Played Track Name: [%s]"
            Text item=spotify_player_xxx_albumName label="Currently Played Album Name: [%s]"
            Text item=spotify_player_xxx_artistName label="Currently Played Artist Name: [%s]"
        }       
    
        Frame label="My Spotify Device 1" {
            Selection item=spotify_device_xxx_3b4....ed4_trackPlay label="Playlist" icon="music" mappings=[
            "spotify:user:spotify:playlist:37i9dQZF1DXdd3gw5QVjt9"="Morning Acoustic",
            "spotify:user:spotify:playlist:37i9dQZEVXcMncpo9bdBsj"="Discover Weekly",
            ]
            Text item=spotify_device_xxx_3b4...ed4_deviceName label="Device Name [%s]"
            Switch item=device1Play
            Slider item=spotify_device_xxx_3b4...ed4_deviceVolume
            Switch item=spotify_device_xxx_3b4...ed4_deviceShuffle
        }    
    
        Frame label="My Spotify Device 2" {
            Selection item=spotify_device_xxx_abc....123_trackPlay label="Playlist" icon="music" mappings=[
            "spotify:user:spotify:playlist:37i9dQZF1DXdd3gw5QVjt9"="Morning Acoustic",
            "spotify:user:spotify:playlist:37i9dQZEVXcMncpo9bdBsj"="Discover Weekly",
            ]
            Text item=spotify_device_xxx_abc...123_deviceName label="Device Name [%s]"
            Switch item=device2Play
            Slider item=spotify_device_xxx_abc...123_deviceVolume
            Switch item=spotify_device_xxx_abc...123_deviceShuffle
        }    
    
    }



