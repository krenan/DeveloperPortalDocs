---
layout: page
title: Android Download-to-Go Library

subcat: Android
weight: 500
---

# DRAFT: Download to Go - Android

> Note: this document is still a draft.

Download to Go (DTG) is an Android library facilitating the download of ABR assets - in particular, DASH and HLS.

The library is external to the Player SDK, and is added separately by the application.
 
Following are some basic sequence diagrams. More will be added soon. 

## New Download 

{% plantuml %}

    @startuml
    
    participant "app: Application" as app
    participant "cm: ContentManager" as cm
    participant "item: DownloadItem" as item
    
    activate app
    activate cm
    
    note over app: User enters media info page
    
    note over app: Check if item exists
    app->cm: findItem(itemId)
    cm->cm: lookup(itemId)
    
    alt item found
        cm-->app: item
    else not found
        cm-->app: null
        app->cm: createItem(itemId, contentURL)
        cm->item: new(itemId, contentURL)
        activate item
        cm-->app: item
    
        app->cm: loadMetadata()
        note over cm
            Download and parse manifest, save in db
        end note
        cm-->app: onTracksAvailable
        cm-->app: onDownloadMetadata
        note over app: * See //track-selection// flow
    end group
    
    note over app: app is ready to start downloading
    app->item: startDownload()
    
    
    @enduml

{% endplantuml %}

## Track Selection

Tracks are selected, by type, using a TrackSelector object. A TrackSelector is obtained by calling `getTrackSelector()` on a DownloadItem.
Given a TrackSelector, the application performs selection:

```java
List<DownloadItem.Track> tracks = trackSelector.getAvailableTracks(AUDIO);
// Application logic for track filtering.
trackSelector.setSelectedTracks(AUDIO, filteredTracks)
// Repeat for other track types (VIDEO, TEXT)

trackSelector.apply();
```


{% plantuml %}

    @startuml
    
    participant "app: App" as app
    participant "item: DownloadItem" as item
    participant "selector: TrackSelector" as selector
    
    group for type in VIDEO, AUDIO, TEXT
        app->selector: tracks = getAvailableTracks(type)
        note over app: filter tracks
        app->selector: setSelectedTracks(type, filteredTracks)
    end group
    app->selector: apply()
    
    @enduml

{% endplantuml %}


### Preference-based Selection

The application should have a policy on track selection, for example:
- Always choose the highest quality video
- Always prefer audio in the user's UI language

The ContentManager calls the application's listener method onTracksAvailable() with an open TrackSelector. The application selects tracks to download and **when onTracksAvailable() returns**, the selection is applied.

If no selection was made, some defaults are applied instead. However, it's recommended that the application adopts its own defaults.


### Interactive Selection Before Download is Started

In this scenario, the metadata for an item was downloaded, the default tracks were selected, but the download wasn't yet started. 
The application calls item.getTrackSelector(), makes a selection and applies it. The selected tracks are downloaded as part of the normal download.



### Interactive Selection After Download is Finished

The entire item has downloaded. The user now decides to download additional tracks - such as another audio language. 
The application calls item.getTrackSelector(), makes a selection and applies it. Then, item.startDownload() is called again, to start downloading the extra tracks.

