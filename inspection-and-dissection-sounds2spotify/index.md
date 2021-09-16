# Inspection and Dissection: Sounds2Spotify


> Available on [Github](https://github.com/LukeBriggsDev/Sounds2Spotify)

# Brief Description
Sounds2Spotify is a web extension for firefox and chrome that converts the tracklists that appear on [BBC Sounds](https://bbc.co.uk/sounds) programme pages to be converted into Spotify playlists.

Unlike [Calmer-Internet](/calmer-internet) this extension can only be installed by following the instructions on [Github](https://github.com/LukeBriggsDev/Sounds2Spotify) due to its use of API keys.

# Impetus
One of my pleasures in life is listening to Music.
My favourite tracks are stored locally and also in Spotify playlists for when I'm using a device without my collection.
Most of the time, however, I'm not listening to my favourite tracks, I'm listening to the [BBC 6Music](https://www.bbc.co.uk/sounds/play/live:bbc_6music).
More specifically, I'm listening to the [Lauren Laverne Breakfast Show](https://www.bbc.co.uk/programmes/b00c000j).

Due to licensing reasons the BBC can only have its music programming available on its own BBC Sounds app for a limited time.
This is fine when I'm using my computer or phone, but quite often I want to listen to music in the background when using my PlayStation, which only allows music to be played through Spotify.

The goal: to convert the music played on BBC Sounds to Spotify playlists.

# Code Walkthrough

## Part 1: Getting the Music
![BBC Sounds Tracklist](/static/postimages/inspection-and-dissection-sounds2spotify/tracklist.png)

<hr style="border: 1px solid black">

![Track popup](/static/postimages/inspection-and-dissection-sounds2spotify/track_popup.png)

Thankfully, underneath every music programme, the BBC provides a track list of all the music played throughout the programme as well as a popover menu providing links to external sources.

So the first job my extension had to do was to scrape this information.
There are no APIs or direct links to this information so the only way to get this information is by clicking on each button and grabbing the Spotify link.

```javascript
// List of buttons that bring up track popover
var list = document.querySelectorAll('[id^="track-"]')
var tracklist = {}
var episodeDate = document.getElementsByClassName("sc-c-episode__metadata__data")[0].lastChild.data
```

Each menu button has the HTML ID of "track-x" where x is its position in the tracklist.
So a query for all these IDs will yield a list of button elements.

The date the episode aired is also quite useful when it comes to naming the eventual playlist, so I grab that as well.

I also initialise the tracklist here. Eventually I want a list of the Spotify links for the tracks.

The IDs and element classes were obtained by hand by scouring the HTML source of the page.

```javascript
for (var i=0; i < list.length; i++){
    list[i].scrollIntoView()
    list[i].click()
    await sleep(t) // t = 200ms
    try{
        var track = document.querySelectorAll('[href^="https://open.spotify.com/track/"]')[0]
        var trackMetadata = JSON.parse(track.getAttribute("data-bbc-metadata"))
        var trackTitle = trackMetadata.TID
        tracklist[trackTitle] = track.getAttribute("href")
    } catch {
        // No links found
    }
}
```

I loop through all the button elements, scroll to the element and click it. There is a 200ms delay between each track as render differences can lead to buttons not being clicked.

When a track is clicked, single popover appears containing a Spotify track link if it exists. As a result, by querying for a Spotify track URL on the page, I will get the link, if it exists.

The popover also includes some metadata for the track in the form of JSON. So I use the track title as the key in a dictionary of tracks, just to make debugging easier. 
The value in the dictionary is the corresponding link.

```javascript
chrome.runtime.sendMessage({type: "tracklist" , track_list: tracklist, name: document.title, date: episodeDate}, function (response) {
    console.log(response.farewell);
})
```

All this data is then wrapped up into a message and sent off to be picked up by a background script.

I mention background scripts in more detail on my [Inspection and Dissection on Calmer-Internet](/inspection-and-dissection-calmer-internet/)

## Part 2: Promises, Async, APIs

Now for the complicated part.
Using the list of Spotify links I have to create a playlist.

```javascript
// Base URL
get_url = "https://accounts.spotify.com/authorize?"
get_url += `client_id=${client_id}&`
get_url += "response_type=code&"
get_url += `redirect_uri=${encodeURIComponent(chrome.identity.getRedirectURL())}&`
get_url += "scope=playlist-modify-public"
```

The first thing I need to do is get authorisation from the user to allow the extension to modify public playlists.
This is done by pointing a request to a specific url containing certain information.

- client_id: this is and id associated with a [Spotify app](https://developer.spotify.com/dashboard/applications).
- response_type: there are various ways to authenticate, the way I are doing it is through a 'code' flow
- redirect_uri: the url Spotify should redirect to when done. 
Web extensions require this to be a single link unique to the extension which is obtained through the method above
- scope: which permissions I require

```javascript
chrome.runtime.onMessage.addListener(
    function(request, sender, sendResponse) {
        if (request.type == "tracklist"){
            sendResponse({farewell: "recieved tracklist"});

            tracklist = request.track_list
            tracklist_name = request.name
            tracklist_date = request.date

            // Authenticate and make playlist
            chrome.identity.launchWebAuthFlow(
                {
                    url: get_url,
                    interactive: true,
                }, authenticateSpotify)
        }
    }
);
```

This is a listener that responds to my message containing a tracklist.
All the listener does is store the information in global variables (I know it can be bad practice but this is a small program with a specific purpose) and launch an authentication flow with my link.

Once this is done, I then call the `authenticateSpotify` method.

```javascript
function authenticateSpotify(response){
    // Get parameters from response
    var urlParams = new URLSearchParams(response.replace(chrome.identity.getRedirectURL(), ''))
        //...
}
```

My WebAuthFlow passes a response in the form of a URL containing a parameter with an authorisation code (or a rejection).
I isolate these parameters by stripping off My redirect url and calling `URLSearchParams` on it.

```javascript
    // POST request for access token
    var authRequest = new Request("https://accounts.spotify.com/api/token",
    {
        method: "POST",
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded;charset=UTF-8'
        },
        body: new URLSearchParams(
        { 
        'grant_type': 'authorization_code', 
        'code': urlParams.get("code"), 
        'redirect_uri': chrome.identity.getRedirectURL(),
        'client_id': `${client_id}`,
        'client_secret': `${client_secret}`
        }) 
    })
```

I then construct a POST request to Spotify by exchanging my authorisation code, and client details for an access token.

```javascript
    // Create playlist
    fetch(authRequest).then(function(response){
        response.json().then(function(json){
            createPlaylist(json.access_token)
        })
    })
```

I then send this POST request, receive a response, parse the JSON response and send the access token to the `createPlaylist` function.

```javascript
async function getSpotifyID(access_token){
    var userRequest = new Request("https://api.spotify.com/v1/me",
    {
        method: "GET",
        headers: {
            Authorization: `Bearer ${access_token}`
        }
    })
    const response = await fetch(userRequest)
    const json = await response.json()
    return json.id
}
```

To know which account I'm working with I ask for the profile ID.

```javascript
async function createPlaylist(access_token){
    var trackURIS = []

    // Add create Spotify URIs from track links
    for (var track of Object.values(tracklist)){
        trackURIS.push(`"${track.replace("https://open.spotify.com/track/", "spotify:track:")}"`)
    }
}
```

The Spotify API does not want the tracks in the form of a link it wants a uri in the form `spotify:track:trackid`.
So for each link in our track list I replace the link part with `spotify:track:`

```javascript
    // Create new playlist
    var request = new Request(`https://api.spotify.com/v1/users/${id}/playlists`,
    {
        method: "POST",
        headers: {
            "Authorization": `Bearer ${access_token}`,
            "Content-Type": "application/json"
        },
        body: 
`{
    "name": "${tracklist_name} | ${tracklist_date}",
    "description": "Made using Sounds2Spotify"
}`
    })
```

Before I can add to a playlist, I need to create one first, so I construct a request to do just that using the authorisation token, and the user ID.
I use the page name, and date from the BBC Sounds page as the title of the playlist.

```javascript
    // Add tracklist songs to newly created playlist
    var request = new Request(`https://api.spotify.com/v1/playlists/${playlistID}/tracks`,
    {
        method: "POST",
        headers: {
            "Content-Type": "application/json",
            Authorization: `Bearer ${access_token}`
        },
        body: 
`{
    "uris": [${trackURIS}]
}`
    })

    var response = await fetch(request)
```

The response of my request contains the playlist ID which I then feed into our next request to add my list of tracks to the playlist.

```javascript
    // Open tab to new playlist
    chrome.tabs.create({ url: `https://open.spotify.com/playlist/${playlistID}`})
```

Finally, I open up the playlist in a new tab.

# Retrospective
Overall, I am very pleased with what I managed to achieve. 
I have very limited experience in using JavaScript to interact with REST APIs, but I managed to pick it up reasonably quickly.

The code I think is relatively clean for a first attempt.
Obviously if any of the request throw an error then there is not much in the way of handling a response, but JavaScript tends to be pretty robust with errors and this isn't a product that is going to be released on any web store or anything.

# Reason for Installation Procedure
The reason why this project is not going to be released on a web store is for security reasons.
A part of the process requires sending Spotify secret API keys for the corresponding Spotify app.
This would mean that my secret keys would have to be in the client-side code, and therefore be freely accessible.

It would therefore be possible for someone to pose as my extension and start making requests as my app to users.

The installation process isn't so cumbersome to be impossible to install, but it isn't as plug and play as I would like it to be

# Sources of Information
- Spotify API Docs - <https://developer.spotify.com/documentation/web-api/>
  - [Playlists](https://developer.spotify.com/console/playlists/)
- Chrome extension API Docs - <https://developer.chrome.com/docs/extensions/reference/>
  - [identity](https://developer.chrome.com/docs/extensions/reference/identity/#method-getRedirectURL)
  - [Message Passing](https://developer.chrome.com/docs/extensions/mv2/messaging/)
- Mozilla Web API Docs - <https://developer.mozilla.org/en-US/docs/Web/API>
  - [fetch()](https://developer.mozilla.org/en-US/docs/Web/API/fetch)
  - [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)
  - [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
