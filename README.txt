# api-client-spotify
----------------------------------------------------------- 1. The Workflow ----------------------------------------------------------
The process is divided into three major steps:

[Your Client] ---> 1. Authentication (Client Credentials) ---> [Spotify Accounts API]
      ^                                                              |
      | <------------- Receives Access Token ------------------------v
      |
[Your Client] ---> 2. Search/Song ID + Token -----------------> [Spotify Web API]
      ^                                                              |
      | <------------- Receives JSON with Metadata ------------------v
      |
[Your Client] ---> 3. Parse JSON and display Info

------------------------------------------------------ 2. Step-by-Step Implementation ------------------------------------------------
Step 1: Register Your Application
	First, you need credentials.

	Create an account on the Spotify Developer Dashboard: You need a standard Spotify account and an email address for validation.
	Once logged in, complete a form to create the app.

	Enter a name and description, as well as the 'Redirect URI' field (for the authentication flow).

	Accept the terms to obtain your credentials (Client ID and Client Secret).

Step 2: Authentication (Obtaining the Access Token)
	The Spotify API is not public; it requires me to identify myself.

Endpoint: POST https://accounts.spotify.com/api/token

Required Headers:
Content-Type: application/x-www-form-urlencoded
Authorization: Basic <Base64(client_id:client_secret)>
Body: grant_type=client_credentials

---------------------------------------------------------------------------------------------------------------------------
Spotify Policy
Low-level detail: we need to implement a function to encode the credentials in Base64 before sending them in the header.

Base64 is an encoding system that takes binary data and transforms it into a text string using a 64-character alphabet.
(Recommendation)
Base64 is NOT encryption; its sole purpose is data formatting. Therefore, this request must be made over HTTPS (encrypted).
---------------------------------------------------------------------------------------------------------------------------

The response will be a JSON containing an access_token. This token lasts for 1 hour and will be used for the next step.

-------------------------------------------------------- Step 3: Querying Song Metadata ----------------------------------------------
Once I have the token, I can go look for the song.
If I have the song ID, I use the Tracks endpoint. Otherwise, I must use the Search endpoint first.

Fetching a track directly

Endpoint: GET [https://api.spotify.com/v1/tracks/](https://api.spotify.com/v1/tracks/){song_id}
Required Headers: Authorization: Bearer <your_access_token>

If you don't have the ID and need to search
Request Structure:

HTTP Method: GET
Base URL: [https://api.spotify.com/v1/search](https://api.spotify.com/v1/search)
Required Headers: Authorization: Bearer <your_access_token>

Since it is a GET method, parameters do not go in the body; instead, they are appended directly to the URL after a question mark (?).

You need to configure three key parameters: query, type, and limit (how many results I want back).

Example URL:
GET [https://api.spotify.com/v1/search?q=comfortably%20numb%20artist:pink%20floyd&type=track&limit=1](https://api.spotify.com/v1/search?q=comfortably%20numb%20artist:pink%20floyd&type=track&limit=1)

Note: %20 is the standard URL encoding replacement for spaces.

Spotify Response: If the request is successful (HTTP Code 200 OK), Spotify will return a JSON. 
Since I set limit=1, the structure of what will be received looks like this:

JSON
{
  "tracks": {
    "items": [
      {
        "name": "Comfortably Numb",
        "id": "58clG7H6fXmZ5OEXb1H6zD",
        "duration_ms": 382293,
        "artists": [
          {
            "name": "Pink Floyd"
          }
        ],
        "album": {
          "name": "The Wall"
        }
      }
    ]
  }
}

-------------------------------------------------------- Step 4: Parsing the JSON ----------------------------------------------------

Spotify will return text in JSON format with all the information (name, artist, album, duration, popularity, etc.).
A module will be implemented to parse this JSON.
