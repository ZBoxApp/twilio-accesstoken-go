# Go Library for Twilio Access Token

Go AccessToken creator for Twilio with support for grants. This library supports HS256, HS384, and HS512 hashing using
a minimal JWT implementation.

Uses the static resources from: https://github.com/TwilioDevEd/video-quickstart-node

This is basically a port of the AccessToken portion of: https://github.com/twilio/twilio-node

### Usage

Install the package

go get github.com/ZBoxApp/twilio-accesstoken-go

### Sample
Sample app using credentials stored in your env

```go
    package main

    import (
	    "encoding/json"
	    "fmt"
	    "github.com/ZBoxApp/twilio-accesstoken-go"
	    "net/http"
	    "os"
    )

    //Convenience interface for printing anonymous JSON objects
    type Response map[string]interface{}

    func (r Response) String() (s string) {
        b, err := json.Marshal(r)
        if err != nil {
            s = ""
        	return
        }

        s = string(b)
        return
    }

    func main() {
    	http.Handle("/", http.FileServer(http.Dir("./static")))
    	http.HandleFunc("/token", token)
    	http.ListenAndServe(":8080", nil)
    }

    // Handles POST request for token
    func token(w http.ResponseWriter, r *http.Request) {
    	if r.Method != "POST" {
    		return
    	}

    	// get credentials for environment variables
    	accountSid := os.Getenv("TW_ACCOUNT_SID")
    	apiKey := os.Getenv("TW_API_KEY")
    	apiSecret := os.Getenv("TW_API_SECRET")

    	// Create an Access Token
    	myToken := accesstoken.New(accountSid, apiKey, apiSecret)

    	// Set the Identity of this token
    	id := "gotwilio.sample"
    	myToken.Identity = id

    	// Grant access to Conversations
    	grant := accesstoken.NewConversationsGrant(os.Getenv("TW_VIDEO_SID"))
    	myToken.AddGrant(grant)

    	signedJWT, err := myToken.ToJWT(accesstoken.DefaultAlgorithm)

    	if err != nil {
    		w.WriteHeader(500)
    		w.Write([]byte(fmt.Sprintf("Error: %v", err)))
    		return
    	}

    	resp := &Response{
    		"identity": id,
    		"token":    signedJWT,
    	}

    	b, err := json.Marshal(resp)
    	if err != nil {
    		w.WriteHeader(500)
    		w.Write([]byte(fmt.Sprintf("Error: %v", err)))
    		return
    	}

    	// Return token info as JSON
    	w.WriteHeader(200)
    	w.Write(b)
    }
```