# Flic Cloud Buttons Documentation

Flic Cloud Buttons is a No-Code integration for Flic buttons into your cloud based service.
The button is connected to a phone or a Flic Hub using the Flic app and will immediately by recognized as a cloud button, with custom appearence and a fully customizable webview.
Button push events are securely delivered to a specified endpoint on your choice.

## Flic Device Manager Button Metadata Fields
To turn a Flic button into a Cloud Button simply involves adding a few metadata fields in the [Flic Device Manager](http://dm2.flic.io/).
They can either be added to an individual button or to a button group.
Fields set in a button group can be overridden at button level to allow for additional level of control.

A Flic Cloud button requires four Metadata fields to be set in the form of a JSON string, and one optional:

```
{
  "__fcb_icon": "${icon}",
  "__fcb_webhook": "${webhook}",
  "__fcb_webview": "${webview}",
  "__fcb_action": "${action}",
  "__fcb_name": "${name}" /* optional */
}
```

`__fcb_icon` should be a direct link to an image in the PNG file format no larger than 512x512. Note that the app will cache these images so if the image is updated, the url needs to change to invalidate the cache.\
Feel free to use our [icon template](https://github.com/50ButtonsEach/flic-cloud-buttons-documentation/blob/main/assets/flic_2_icon_template.png).

`__fcb_webhook` determines the endpoint that will be hit when the button is pushed.

`__fcb_webview` is a URL to a webpage that will be displayed inside a webview within the Flic app when a user taps on the button in the button list.

`__fcb_action` can be either `CLOUD` or `CLOUD_LOCATION`.

There is also an optional key, `__fcb_name`, that will set the default name of the button.


In the case of `CLOUD_LOCATION`, the Geolocation of the phone will be attached to the webhook request.
Note that this will only work on mobile devices and if the user has allowed permission to access location data.
If this data is used, it is required that you add a disclaimer explaining how this data will be used in the webview, and instructions on how the user can request deletion of this data.

## JWT
All data that is included with the requests are encoded and secured using [JSON Web Token](https://jwt.io).
JWT offers the reciever the ability to verify the authenticity of the JSON data to ensure that the request is genuine. This provides protection against request forgery.
The payload part of the token will be a JSON structure containing Flic-specific data about the request as well as the standardized "exp" parameter.
Our backend signs all JWTs using the ES256 algorithm.

## Webview Request

The app will supply a query string parameter, `token`, containing a JWT with information about the button that is being displayed.
Example:
`{WEBVIEW_URL}?token=eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9...`

Example webview payload encoded in the JWT:
```
{
    "exp": 1721214346,
    "buttonUuid": "336c33540c134a929824e41c0bd3c377",
    "buttonSerialNumber": "BC44-B06243",
    "buttonBatteryStatus": "high"
}
```

`buttonBatteryStatus` can have the values `unknown`, `low`, `medium` or `high`.

The P-256 public key used to verify JWTs for this type of request is as follows, encoded in various formats:
```
JWK format:
{
    "kty": "EC",
    "x": "qHF1y2ztTcdfknnISSozf-hdoXi6ylzLDGasj7I5TbM",
    "y": "ahj1rHupaOpKviR3Gy4W1abQMJ9AQdG9JvmNvoaZ4XI",
    "crv": "P-256"
}

PEM format:
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEqHF1y2ztTcdfknnISSozf+hdoXi6
ylzLDGasj7I5TbNqGPWse6lo6kq+JHcbLhbVptAwn0BB0b0m+Y2+hpnhcg==
-----END PUBLIC KEY-----

DER format (hex):
3059301306072a8648ce3d020106082a8648ce3d03010703420004a87175cb6c
ed4dc75f9279c8492a337fe85da178baca5ccb0c66ac8fb2394db36a18f5ac7b
a968ea4abe24771b2e16d5a6d0309f4041d1bd26f98dbe8699e172

Raw format (x and y coordinates):
x: 0xA87175CB6CED4DC75F9279C8492A337FE85DA178BACA5CCB0C66AC8FB2394DB3
y: 0x6A18F5AC7BA968EA4ABE24771B2E16D5A6D0309F4041D1BD26F98DBE8699E172
```

## Webhook Request

When the button is pushed, the push event will be proxied through Flic servers to hide the actual endpoint.
It will ultimately be delivered as a POST request to the `WEBHOOK_URL` with a JSON body containing the JWT in the "token" field.
```
POST {WEBHOOK_URL}
Content-Type: application/json

{
  "token": "eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```
Example webhook payload encoded in the JWT:
```
{
    "exp": 1721214346,
    "buttonUuid": "336c33540c134a929824e41c0bd3c377",
    "buttonSerialNumber": "BC44-B06243",
    "buttonBatteryStatus": "high",
    "pressedAt": "2024-07-17T12:05:45+02:00",
    "clickType": "click"
}
```

`buttonBatteryStatus` can have the values `unknown`, `low`, `medium` or `high`.
`clickType` can have the values `click`, `double_click` or `hold`.

The P-256 public key used to verify JWTs for this type of request is as follows, encoded in various formats:
```
JWK format:
{
    "kty": "EC",
    "x": "tOBY_7kJlAPE19qweawQlQah10rOQzhaxfjlen4RasM",
    "y": "wSbZUG_MoKAVtZTDL2ocWRzCjE43ob6i5PoXC8T8jkY",
    "crv": "P-256"
}

PEM format:
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEtOBY/7kJlAPE19qweawQlQah10rO
Qzhaxfjlen4RasPBJtlQb8ygoBW1lMMvahxZHMKMTjehvqLk+hcLxPyORg==
-----END PUBLIC KEY-----

DER format:
3059301306072a8648ce3d020106082a8648ce3d03010703420004b4e058ffb9
099403c4d7dab079ac109506a1d74ace43385ac5f8e57a7e116ac3c126d9506f
cca0a015b594c32f6a1c591cc28c4e37a1bea2e4fa170bc4fc8e46

Raw format (x and y coordinates):
x: 0xB4E058FFB9099403C4D7DAB079AC109506A1D74ACE43385AC5F8E57A7E116AC3
y: 0xC126D9506FCCA0A015B594C32F6A1C591CC28C4E37A1BEA2E4FA170BC4FC8E46
```

## Webhook Response
In response to a webhook request, the server should respond with HTTP status `200 OK`.
Additionally, the server can respond with an optional message and/or sound. This message will be shown as a notification in the app.
```
200 OK
Content-Type: application/json

{
    "message": "Button event received!",
    "sound": "https://example.com/mysound.mp3"
}
```

## Import Buttons to Flic Device Manager
There are two ways to get buttons into the [Flic Device Manager](http://dm2.flic.io/).
You can either use the Flic Hub SDK to manually import the buttons or you can  [order buttons in bulk](https://flic.io/business#contact) from us and have them pre-registered to your account and Cloud Button configuration.

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

