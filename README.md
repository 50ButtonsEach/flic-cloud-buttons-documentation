# Flic Cloud Buttons Documentation




## Flic Device Manager Button Metadata fields
To turn a Flic button into a Cloud Button simply involves adding a few metadata fields in the Device Manager.
They can either be added to an individual button or to a button group.
Fields set in a button group can be overridden at button level to allow for additional level of control.



```
{
  "__fcb_icon": ICON_URL,
  "__fcb_webhook": WEBHOOK_URL
  "__fcb_webview": WEBVIEW_URL
  "__fcb_action": ACTION
}
```

`ICON_URL` should be a direct link to an image in the PNG file format no larger than 512x512. Note that the app will cache these images so if the image is updated, the url needs to change to invalidate the cache.
`WEBHOOK_URL` determines the endpoint that will be hit when the button is pushed.
`WEBVIEW_URL` is a URL to a webpage that will be displayed inside a webview within the Flic app when a user taps on the button in the button list.
`ACTION` can be either `CLOUD` or `CLOUD_LOCATION`
In the case of `CLOUD_LOCATION` the Geolocation of the phone will be attached to the webhook request.
Note that this will only work on mobile devices and if the user has allowed permission for the Flic app to access geolocation.
If this action is used, it is required that you add a disclaimer explaining how this data will be used in the webview, and instructions on how the user can request deletion of this data.

## Webview request
The app will supply a query string parameter, `token`, containing a JWT with information about the button that is being displayed.
Example:
`{WEBHOOK_URL}?token={JWT}`

