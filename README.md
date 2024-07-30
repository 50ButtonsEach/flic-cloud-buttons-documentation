# Flic Cloud Buttons Documentation

Flic Cloud Buttons is a No-Code integration for Flic buttons into your cloud based service.
The button is connected to a phone or a Flic Hub using the Flic app and will immediately by recognized as a cloud button, with custom appearence and a fully customizable webview.
Button push events are securely delivered to a specified endpoint on your choise.

## Flic Device Manager Button Metadata Fields
To turn a Flic button into a Cloud Button simply involves adding a few metadata fields in the [Flic Device Manager](http://dm2.flic.io/).
They can either be added to an individual button or to a button group.
Fields set in a button group can be overridden at button level to allow for additional level of control.

A Flic Cloud button requires these 4 Metadata fields to be set in the form of a JSON string:

```
{
  "__fcb_icon": ICON_URL,
  "__fcb_webhook": WEBHOOK_URL
  "__fcb_webview": WEBVIEW_URL
  "__fcb_action": ACTION
}
```

`ICON_URL` should be a direct link to an image in the PNG file format no larger than 512x512. Note that the app will cache these images so if the image is updated, the url needs to change to invalidate the cache.\
Feel free to use our [icon template](https://github.com/50ButtonsEach/flic-cloud-buttons-documentation/blob/main/assets/flic_2_icon_template.png).

`WEBHOOK_URL` determines the endpoint that will be hit when the button is pushed.

`WEBVIEW_URL` is a URL to a webpage that will be displayed inside a webview within the Flic app when a user taps on the button in the button list.

`ACTION` can be either `CLOUD` or `CLOUD_LOCATION`.

In the case of `CLOUD_LOCATION`, the Geolocation of the phone will be attached to the webhook request.
Note that this will only work on mobile devices and if the user has allowed permission to access location data.
If this data is used, it is required that you add a disclaimer explaining how this data will be used in the webview, and instructions on how the user can request deletion of this data.

## JWT
All data that is included with the requests are encoded using [JSON Web Token](https://jwt.io).
JWT offers the reciever the ability to verify the authenticity of the JSON data to ensure that the request is genuine. This provides protection against request forgery.

## Webview Request

The app will supply a query string parameter, `token`, containing a JWT with information about the button that is being displayed.
Example:
`{WEBVIEW_URL}?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

Example webview payload:
```
{
    "exp": 1721214346,
    "buttonUuid": "336c33540c134a929824e41c0bd3c377",
    "buttonSerialNumber": "BC44-B06243",
    "buttonName": "BC44-B06243",
    "buttonBatteryStatus": "high",
    "pressedAt": "2024-07-17T12:05:45+02:00",
}
```

## Webhook Request

When the button is pushed, the push event will be proxied through Flic servers to hide the actual endpoint.
It will ultimately be delivered as a POST request to the `WEBHOOK_URL` with a JSON containing the JWT under the key "token".
```
POST {WEBHOOK_URL}
Content-Type: application/json

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```
Example webhook payload:
```
{
    "exp": 1721214346,
    "buttonUuid": "336c33540c134a929824e41c0bd3c377",
    "buttonSerialNumber": "BC44-B06243",
    "buttonName": "BC44-B06243",
    "buttonBatteryStatus": "high",
    "pressedAt": "2024-07-17T12:05:45+02:00",
    "clickType": "click"
}
```

## Webhook Response
In response to a webhook the request, the server should respond with HTTP status `200 OK`.
Additionally, the server can respond with an optional message. This message will be shown as a notification in the app.
```
{
    "message": "Button event received!"
}
```

## Import Buttons to Flic Device Manager
There are two ways to get buttons into the [Flic Device Manager](http://dm2.flic.io/).
You can either use the Flic Hub SDK to manually import the buttons or you can [order buttons in bulk](https://flic.io/design-your-flic) from us and have them pre-registered to your account and Cloud Button configuration.

### Manual Import
#### 1. Export Buttons Using Flic Hub SDK
- Connect to your Flic Hub LR using the [Flic Hub SDK](https://hubsdk.flic.io)
- Run the command `scan` in the Command Prompt to start a button scan.
- When the button is connected, run the command `exportbuttons` and copy the pairing data.

#### 2. Import into Flic Device Manager
- In the [Flic Device Manager](http://dm2.flic.io/), go to Buttons → Import Buttons.
- Paste the pairing data, press "Parse", and check the buttons you wish to import and press Import Buttons.

If you want to add a large amount of buttons you can use the `bulkScan` command in the Flic Hub SDK to scan multiple buttons at once.\
Note that you can only connect 63 buttons at a time so to do a large amount of buttons it would be recommended to do them in batches of 50.

