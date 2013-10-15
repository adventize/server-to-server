# Affiliate networks integration documentation

## How you request our advertisement offers

You can request our offers from Adventize servers by HTTP protocol, using GET method. Fetch URL: <http://api.adventize.com/fetch/>.

## Request parameters

 Name         |Type   |Platform|Required
:-------------|:------|:-------|:-------
 `app_id`     |String |Both    |Yes
 `timestamp`  |Integer|Both    |Yes 
 `secret`     |String |Both    |Yes
 `idfa`       |String |iOS     |No
 `android_id` |String |Android |No
 `open_udid`  |String |Both    |Yes
 `udid`       |String |Both    |No
 `imei`       |String |Both    |No
 `mac`        |String |Both    |No
 `serial`     |String |Android |No
 `ip`         |String |Both    |Yes
 `device_type`|String |Both    |Yes
 `internet`   |String |Both    |Yes
 `os_version` |String |Both    |Yes
 `resolution` |String |Both    |Yes
 `extra*`     |String |Both    |Yes

### Parameters description

#### General

`app_id` — Affiliate network ID. Your account will be associated with `app_id` when we register you at Adevntize. To obtain your app_id please contact your manager at Adventize.

`timestamp` — UNIX-timestamp. Used for security purposes.

`secret` — Secret hash. Used for signing your request to prevent fraudulent activity. Please see how it is calculated below.

The formula generates your secret parameter:

    sha1(secret_word + [concatenation of all the request parameters (names and values), sorted by name in alphabetical order]);

Where `secret_word` – is the parameter that you will get from your Adventize account manager.

###### Example

You request Adventize server with following parameter: `app_id` – 3245657866, `timestamp` – 1378182410, `open_udid` – m5f94jf8693hfh, `device_type` – android-tablet.

The secret parameter will be:

    sha1(secret_word + "app_id" + "3245657866" + "device_type" + "android-tablet" + "open_udid" + "m5f94jf8693hfh" + "timestamp" + "1378182410");
    
###### PHP impletemtation

    function generateSecret($secret_word, $params) {
        $keys = array_keys($params);
        sort($keys);
        $sign = $secret_word;
        foreach ($keys as $key) {
            if ($params[$key]) {
                $sign .= $key . $params[$key];
            }
        }
        return sha1($sign);
    }

#### Device identification

`idfa` — Apple's "IDentifier For Advertisers".

`android_id` — The Android ID of the device.

`open_udid` — Universal device identification. Generation algorithm described at <OpenUDID.org>.

`udid` — Apple's deprecated identification. But some applications still can send it.

`imei` — IMEI.

`mac` — MAC-address of network device.

`serial` — The factory serial number.

#### Targeting

`ip` — Client's IP-address.

`device_type` — Type of device. Server accept following values: iphone, ipod, ipad, android, android-tablet, android-phone.

`internet` — Type of internet connection. Two values: `wifi`, `3g`.

`os_version` — Version of client’s OS.

`resolution` — Screen resolution. String "WIDTHxHEIGHT", eg. "1920x1080".

#### Extra parameters

`extra*` — Any parameter that begins with `extra` will be forwarded to the callback that informs you on execution of any of the targeted action (installation, or any other action type). Please see more about your callback in this document bellow.

### Server response

The response is in the JSON format. The successful response will have code 200.

#### Successful response format description

    {
        "data": [
          {   "offer_id": "2bf8c6f620c672db559444c5a232b62f4rth",
              "offer_description": "Install and run",
              "name": "Offer #1",
              "url": "http://api.adventize.com:80/click/?url=620c672db559444c5a232b62f4rth",
              "icon": "http://icons.adventize.com/cache/aeaa0c4b8cc857771bd4acfc06ec62b.png"
          },{ "offer_id": "848wcwtg8976ba419f8632e3dbe63d19erwd",
              "offer_description": "Install and run",
              "name": "Offer #2",
              "url": "http://api.adventize.com:80/click/?url=6f52135c8488976ba419f8632e3dv",
              "icon": "http://icons.adventize.com/cache/ec5aeb08fc71a826bca05cc7d06f521.png"
          }, ...
        ]
    }

Field name         |Description
:------------------|:----------
`offer_id`         |Offer identification
`offer_description`|Offer description
`name`             |Offer's title
`url`              |Link to the advertised entity. If the entity is an application, then this will be a link to the application page in the store
`icon`             |Link to the advertised entity icon

Accordingly, if there is no advertisement to show, the server will response with:

    {"data":[]}
    
#### Error response from the server

The error response from the server is any response with status code other then 200. Status codes of false replies are:

**400** – Bad request. The request was made with wrong parameters. For example, if the idfa parameter is filled correctly, but the device type is “android”. As the idfa can be filled only for iOS device, we assume the request is bad.

**401** – Unauthorized. Server fails to recognize the secret parameter. 

**404** – Not found. The request was made to the URL witch does not exist. 

**410** – Gone. Offer has expired.

**50x** – Adventize server error. We apologize for those.

If the `Content-Type` header is set to `text/html`, then the server will response with proper HTML-page. In all other cases the server will reply in JSON format:

    {"status": "error", "message": "[error message]"}
    
    
### Installation notification

Adventize may notify your service about fulfillment of certain actions (installation, click, other action). In this case you need to give us a proper URL that will be requested by Adventize service every time on when this action is executed. Also there will be an `extra*` parameter in this request (please see the parameter description for details).
