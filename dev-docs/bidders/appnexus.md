---
layout: bidder
title: AppNexus
description: Prebid AppNexus Bidder Adaptor
biddercode: appnexus
media_types: banner, video, native
tcfeu_supported: true
dsa_supported: true
prebid_member: true
userIds: all (with commercial activation)
schain_supported: true
coppa_supported: true
usp_supported: true
gpp_supported: true
floors_supported: true
fpd_supported: false
pbjs: true
pbjs_version_notes: please avoid using v7.15 and v7.16
pbs: true
gvl_id: 32
sidebarType: 1
---

### Table of Contents

- [Table of Contents](#table-of-contents)
  - [Bid Params](#bid-params)
  - [Video Object](#video-object)
  - [User Object](#user-object)
  - [App Object](#app-object)
  - [Video Bid params and Video MediaTypes params](#video-bid-params-and-video-mediatypes-params)
  - [Custom Targeting keys](#custom-targeting-keys)
  - [Auction Level Keywords](#auction-level-keywords)
  - [Passing Keys Without Values](#passing-keys-without-values)
  - [First Party Data](#first-party-data)
  - [User Sync in AMP](#user-sync-in-amp)
  - [Mobile App Display Manager Version](#mobile-app-display-manager-version)
  - [Debug Auction](#debug-auction)
  - [Prebid Server Test Request](#prebid-server-test-request)

<a name="appnexus-bid-params"></a>

{: .alert.alert-danger :}
All AppNexus (Microsoft/Xandr) placements included in a single call to `requestBids` must belong to the same parent Publisher.  If placements from two different publishers are included in the call, the AppNexus bidder will not return any demand for those placements. <br />
*Note: This requirement does not apply to adapters that are [aliasing](/dev-docs/publisher-api-reference/aliasBidder.html) the AppNexus adapter.*

#### Bid Params

{: .alert.alert-danger :}
Starting with Prebid.js version 9.0, an update was made to the `appnexusBidAdapter.js` file to remove the support for the `transformBidParams` function.  Previously this standard adapter function was used in conjunction of Prebid.js > PBS requests to modify any bid params for that bidder to the bid param format used by the PBS endpoint.  Part of the changes for 9.0 in general were to remove these functions from the client-side adapter files, in order to reduce the build size of Prebid.js for those publishers who wanted to make the PBS requests.  In the case of our adapter, we instead created a new module named `anPspParamsConverter` that would mimic behavior of the `transformBidParams` function.  There's no setup instructions needed on the Prebid.js configs, the module only needs to be included in the Prebid.js build file and it will perform the needed steps.  If you have any questions on this change, please reach out to your Microsoft representative and they can help.

{: .alert.alert-danger :}
Starting with Prebid.js version 7.36.0, an update was made to the `appnexusBidAdapter.js` file to support bid params in a lower-case underscore format (eg `invCode` to `inv_code`) similar to how the params are formatted for the Prebid Server AppNexus bidder.  This change was implemented to streamline publisher setups for both projects instead of maintaining separate versions of the same params depending on what setup is used.
To avoid breaking changes, the old 'camelCase' format is still currently supported for all AppNexus bid params in the `appnexusBidAdapter.js` file. If you are using an older version of Prebid.js, you will need to continue to use the older 'camelCase' format as appropriate.
The table below will reflect both formats, though it's recommended to use the lower-case underscore format where possible going forward (assuming you're using a compatible version of Prebid.js).

{: .table .table-bordered .table-striped }
| Name                | Scope    | Description                                                                                                                                                                   | Example                                               | Type             |
|---------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------|------------------|
| `placement_id` (PBS+PBJS) or `placementId` (PBJS)    | required | The placement ID from AppNexus.  You may identify a placement using the `invCode` and `member` instead of a placement ID. This parameter can be either a `string` or `integer` for Prebid.js, however `integer` is preferred. Legacy code can retain the `string` value. **Prebid Server requires an integer value.**                                                    | `234234`                                            | `integer`         |
| `member`                                        | optional | The member ID  from AppNexus. Must be used with `invCode`.                                                                                                                    | `'12345'`                                             | `string`         |
| `invCode` or `inv_code`                             | optional | The inventory code from AppNexus. Must be used with `member`.                                                                                                                 | `'abc123'`                                            | `string`         |
| `publisherId` or `publisher_id`                    | optional | The publisher ID from AppNexus. It is used by the AppNexus end point to identify the publisher when placement id is not provided and `invCode` goes wrong. The `publisherId` parameter can be either a `string` or `integer` for Prebid.js, however `integer` is preferred.                                                                                                                    | `12345`                                             | `integer`         |
| `frameworks`                                    | optional | Array of integers listing API frameworks for Banner supported by the publisher. | `[1,2]` | `array of integer` |
| `user`                                          | optional | Object that specifies information about an external user. See [User Object](#appnexus-user-object) for details.                                                               | `user: { age: 25, gender: 0, dnt: true}`              | `object`         |
| `allowSmallerSizes` or `allow_smaller_sizes`               | optional | If `true`, ads smaller than the values in your ad unit's `sizes` array will be allowed to serve. Defaults to `false`.                                                         | `true`                                                | `boolean`        |
| `usePaymentRule` (PBJS) or `use_pmt_rule` (PBS+PBJS) | optional | If `true`, Appnexus will return net price to Prebid.js after publisher payment rules have been applied.                                                                       | `true`                                                | `boolean`        |
| `keywords`                                      | optional | A set of key-value pairs applied to all ad slots on the page.  Mapped to [buy-side segment targeting](https://learn.microsoft.com/en-us/xandr/monetize/segment-targeting) (login required). A maximum of 100 key/value pairs can be defined at the page level. Each tag can have up to 100 additional key/value pairs defined. Values can be empty. See [Passing Keys Without Values](#appnexus-no-value) below for examples. If you want to pass keywords for all adUnits, see [Auction Level Keywords](#appnexus-auction-keywords) for an example. Note that to use keyword with the Prebid Server adapter, that feature must be enabled for your account by an AppNexus account manager. | `keywords: { genre: ['rock', 'pop'] }`                | `object`         |
| `video`                                         | optional | Object containing video targeting parameters.  See [Video Object](#appnexus-video-object) for details.                                                                        | `video: { playback_method: ['auto_play_sound_off'] }` | `object`         |
| `app`                                           | optional | Object containing mobile app parameters.  See the [App Object](#appnexus-app-object) for details.                                                                      | `app : { id: 'app-id'}`                               | `object`         |
| `reserve`                                       | optional | Sets a floor price for the bid that is returned. If floors have been configured in the AppNexus Console, those settings will override what is configured here unless 'Reserve Price Override' is checked. See [Microsoft Learn](https://learn.microsoft.com/en-us/xandr/monetize/create-a-floor-rule)                | `0.90`                                                | `float`          |
| `position`                                      | optional | Identify the placement as above or below the fold.  Allowed values: Unknown: `unknown`; Above the fold: `above`; Below the fold: `below`                                      | `'above'`                                               | `string`        |
| `trafficSourceCode` or `traffic_source_code`         | optional | Specifies the third-party source of this impression.                                                                                                                          | `'my_traffic_source'`                                 | `string`         |
| `supplyType` or `supply_type`                      | optional | Indicates the type of supply for this placement. Possible values are `web`, `mobile_web`, `mobile_app`                                                                        | `'web'`                                               | `string`         |
| `pubClick` or `pub_click`                         | optional | Specifies a publisher-supplied URL for third-party click tracking. This is just a placeholder into which the publisher can insert their own click tracker. This parameter should be used for an unencoded tracker. This parameter is expected to be the last parameter in the URL. Please note that the click tracker placed in this parameter will only fire if the creative winning the auction is using AppNexus click tracking properly.                                  | `'http://click.adserver.com/'`                        | `string`         |
| `extInvCode` or `ext_inv_code`                      | optional | Specifies predefined value passed on the query string that can be used in reporting. The value must be entered into the system before it is logged.                           | `'10039'`                                             | `string`         |
| `externalImpId` or `external_imp_id`                   | optional | Specifies the unique identifier of an externally generated auction.                                                                                                           | `'bacbab02626452b097f6030b3c89ac05'`                  | `string`         |
| `generate_ad_pod_id`                            | optional | Signal to AppNexus to split impressions by ad pod and add unique ad pod id to each request. Specific to long form video endpoint only. Supported by Prebid Server, not Prebid JS.  | `true`                                                | `boolean`        |

<a name="appnexus-video-object"></a>

#### Video Object

For details on how these video params work with the params set in the adUnit.mediaTypes.video object, see [Video Bid params and Video MediaTypes params](#appnexus-video-params) section below.

{: .table .table-bordered .table-striped }
| Name              | Description                                                                                                                                                                                                                                  | Type             |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------|
| `minduration` | Integer that defines the minimum video ad duration in seconds. | `integer` |
| `maxduration` | Integer that defines the maximum video ad duration in seconds. | `integer` |
|`context` | A string that indicates the type of video ad requested.  Allowed values: `"pre_roll"`; `"mid_roll"`; `"post_roll"`; `"outstream"`; `"in-banner"`, `"in-feed"`, `"interstitial"`, `"accompanying_content_pre_roll"`, `"accompanying_content_mid_roll"`, `"accompanying_content_post_roll"`. | `string` |
| `skippable` | Boolean which, if `true`, means the user can click a button to skip the video ad.  Defaults to `false`. | `boolean` |
|`skipoffset`| Integer that defines the number of seconds until an ad can be skipped.  Assumes `skippable` setting was set to `true`. | `integer` |
| `playback_method` | A string that sets the playback method supported by the publisher.  Allowed values: `"auto_play_sound_on"`; `"auto_play_sound_off"`; `"click_to_play"`; `"mouse_over"`; `"auto_play_sound_unknown"`. | `string` |
| `frameworks` | Array of integers listing API frameworks supported by the publisher.  Allowed values: None: `0`; VPAID 1.0: `1`; VPAID 2.0: `2`; MRAID 1.0: `3`; MRAID 2.0: `4`; ORMMA: `5`; OMID 1.0 `6`. | `Array<integer>` |

<a name="appnexus-user-object"></a>

#### User Object

{: .table .table-bordered .table-striped }
| Name              | Description                                                                                                                     | Example                                                                  | Type             |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|------------------|
| `age`             | The age of the user.                                                                                                            | `35`                                                                     | `integer`        |
| `externalUid` or `external_uid`    | Specifies a string that corresponds to an external user ID for this user.                                                       | `'1234567890abcdefg'`                                                    | `string`         |
| `segments`        | Specifies the segments to which the user belongs.                                                                               | `[1, 2]`                                                                 | `Array<integer>` |
| `gender`          | Specifies the gender of the user.  Allowed values: Unknown: `0`; Male: `1`; Female: `2`                                         | `1`                                                                      | `integer`        |
| `dnt`             | Do not track flag.  Indicates if tracking cookies should be disabled for this auction                                           | `true`                                                                   | `boolean`        |
| `language`        | Two-letter ANSI code for this user's language.                                                                                  | `EN`                                                                     | `string`         |

<a name="appnexus-app-object"></a>

#### App Object

AppNexus supports using prebid within a mobile app's webview. If you are interested in using an SDK, please see [Prebid Mobile]({{site.baseurl}}/prebid-mobile/prebid-mobile.html) instead.

{: .table .table-bordered .table-striped }
| Name              | Description                                                                                                                     | Example                                                                  | Type             |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------|------------------|
| `id`              | The App ID.                                                                                                                     | `'B1O2W3M4AN.com.prebid.webview'`                                        | `string`         |
| `device_id`       | Object that contains the advertising identifiers of the user (`idfa`, `aaid`, `md5udid`, `sha1udid`, or `windowsadid`).         | `{ aaid: "38400000-8cf0-11bd-b23e-10b96e40000d" }`                       | `object`         |
| `geo`             | Object that contains the latitude (`lat`) and longitude (`lng`) of the user.                                                    | `{ lat: 40.0964439, lng: -75.3009142 }`                                  | `object`         |

<a name="custom-targeting-keys"></a>

#### Custom Targeting keys

AppNexus returns custom keys that can be sent to the adserver through bidderSettings: buyerMemberId, dealPriority, and dealCode. The following snippet demonstrates how to add these custom keys as key-value pairs.

```javascript
pbjs.bidderSettings = {
  appnexus: {
    adserverTargeting: [
      {
        key: "apn_buyer_memberid", // Use key configured in your adserver
        val: function(bidResponse) {
          return bidResponse.appnexus.buyerMemberId;
        }
      },
      {
        key: "apn_prio", // Use key configured in your adserver
        val: function(bidResponse) {
          return bidResponse.appnexus.dealPriority;
        }
      }, {
        key: "apn_dealcode", // Use key configured in your adserver
        val: function(bidResponse) {
          return bidResponse.appnexus.dealCode;
        }
      }
    ]
  }
}
```

<a name="appnexus-video-params"></a>

#### Video Bid params and Video MediaTypes params

It is possible to have setup video params within the adUnit's AppNexus bid object as well as in the adUnit's `mediaTypes.video` object.  In this case, there is a set of logic that the AppNexus bid adapter follows to resolve which values should be passed along to the ad server request.  Generally speaking, the adapter prefers the values from bid param video object over the mediaTypes video object, in order to preserve historical setups.  So for instance, if the playbackmethod field was set in both locations, then the bid params `playback_method` would be chosen over the mediaTypes `playbackmethod`.  If there are different fields set between the two locations and they don't overlap, then the `mediaTypes.video` params would be included along with the bid params.

To note - not all the fields between the two locations have a single direct comparison, nor are all fields from the `mediaTypes.video` object are supported by the ad server endpoint.  The fields/values set in the `mediaTypes.video` object follow the OpenRTB convention, while our bid params follow the convention for the ad server endpoint (which is **not** a straight OpenRTB endpoint).  The AppNexus bid adapter converts the matching fields from the `mediaTypes.video` object where there is a correlation to help support as much as possible.  For example, to help infer the `context` field, the adapter will look to the `mediaTypes.video.plcmt`, `mediaTypes.video.startdelay`, and `mediaTypes.video.placement` fields to help determine the appropriate `context` value.  The `startdelay` field is included here to help clarify the type of instream video that is used (ie pre/mid/post-roll).  

If you want to transition from video bid params to use the `mediaTypes.video` params (to simplify the adUnit setup), please contact your AppNexus contact to help identify the proper fields/values are populated to ensure a smooth transition.

<a name="appnexus-auction-keywords"></a>

#### Auction Level Keywords

It's possible to pass a set of keywords for the whole request, rather than a particular adUnit.  Though they would apply to all adUnits (which include the appnexus bidder) in an auction, these keywords can work together with the bidder level keywords (if for example you want to have specific targeting for a particular adUnit).

Below is an example of how to define these auction level keywords for the appnexus bidder:

```javascript
pbjs.setConfig({
  appnexusAuctionKeywords: {
    genre: ['classical', 'jazz'],
    instrument: 'piano'
  }
});
```

Like in the bidder.params.keywords, the values here can be empty.  Please see the section immediately below for more details.

<a name="appnexus-no-value"></a>

#### Passing Keys Without Values

It's possible to use the `keywords` parameter to define keys that do not have any associated values. Keys with empty values can be created in Prebid.js and can also be sent through Prebid Server to AppNexus. The following are examples of sending keys with empty values:

```javascript
keywords: {
  myKeyword: '',
  myOtherKeyword: ['']
}
```

The preceding example passes the key `myKeyword` with an empty value. The key `myOtherKeyword` contains an empty value array.

You can define keys with values and without values in the same `keywords` definition. In this next example, we've defined the key `color` with an array of values: `red`, `blue`, and `green`. We've followed that with the key `otherKeyword` with an empty value array.

```javascript
keywords: {
  color: ['red', 'blue', 'green'],
  otherKeyword: ['']
}
```

<a name="appnexus-fpd"></a>

#### First Party Data

Publishers should use the `ortb2` method of setting [First Party Data](https://docs.prebid.org/features/firstPartyData.html).

At this time however, the `appnexus` bidder fully reads the First Party Data when using the Prebid Server and Prebid Server Premium endpoints.  The client-side version of the `appnexus` bidder has partial support to read all the various keywords parameters from the First Party Data fields.  There is also some special support with the segment fields but only from known sources which are specifically configured.  All other First Party Data fields are not read at this time.

PBS/PSP supports all first party data fields: site, user, segments, and imp-level first party data.

<a name="appnexus-amp"></a>

#### User Sync in AMP

If you are syncing user id's with Prebid Server and are using AppNexus' managed service, see [AMP Implementation Guide cookie-sync instructions](/dev-docs/show-prebid-ads-on-amp-pages.html#user-sync) for details.

<a name="appnexus-debug-auction"></a>

#### Mobile App Display Manager Version

The AppNexus endpoint expects `imp.displaymanagerver` to be populated for mobile app sources
requests, however not all SDKs will populate this field. If the `imp.displaymanagerver` field
is not supplied for an `imp`, but `request.app.ext.prebid.source`
and `request.app.ext.prebid.version` are supplied, the adapter will fill in a value for
`diplaymanagerver`. It will concatenate the two `app` fields as `<source>-<version>` fo fill in
the empty `displaymanagerver` before sending the request to AppNexus.

#### Debug Auction

{: .alert.alert-danger :}
Enabling the AppNexus Debug Auction feature should only be done for diagnosing the AppNexus auction. Do not enable this feature in a production setting where it may impact users.

To understand what is happening behind the scenes during an auction, you can enable a debug auction by adding an `apn_prebid_debug` cookie with a JSON string. For example:

```javascript
{ "enabled": true, "dongle": "QWERTY", "debug_timeout": 1000, "member_id": 958 }
```

To view the results of the debug auction, add the `pbjs_debug=true` query string parameter and open your browser's developer console.

{: .table .table-bordered .table-striped }
| Name              | Description                                                     | Example               | Type             |
|-------------------|-----------------------------------------------------------------|-----------------------|------------------|
| `enabled`         | Toggle the debug auction to occur                               | `true`                | `boolean`        |
| `dongle`          | Your account's unique debug password.                           | `QWERTY`              | `string`         |
| `member_id`       | The ID of the member running the debug auction                  | `958`                 | `integer`        |
| `debug_timeout`   | The timeout for the debug auction results to be returned        | `3000`                | `integer`        |

#### Prebid Server Test Request

The following test parameters can be used to verify that Prebid Server is working properly with the
server-side Appnexus adapter. This example includes an `imp` object with an Appnexus test placement ID and sizes
that would match with the test creative.

```json
"imp": [{
  "id": "some-impression-id",
  "banner": {
    "format": [{
      "w": 600,
      "h": 500
    }, {
      "w": 300,
      "h": 600
    }]
  },
  "ext": {
    "appnexus": {
      "placement_id": 13144370
    }
  }
}]
```
