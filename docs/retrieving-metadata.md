---
title: Retrieving metadata
---

This documentation may be out-of-date. Please refer to the
[documentation for the latest ExoPlayer release][] on developer.android.com.
{:.info}

## During playback ##

The metadata of the media can be retrieved during playback in multiple ways. The
most straightforward is to listen for the
`Player.Listener#onMediaMetadataChanged` event; this will provide a
[`MediaMetadata`][] object for use, which has fields such as `title` and
`albumArtist`. Alternatively, calling `Player#getMediaMetadata` returns the same
object.

~~~
public void onMediaMetadataChanged(MediaMetadata mediaMetadata) {
  if (mediaMetadata.title != null) {
    handleTitle(mediaMetadata.title);
  }
}

~~~
{: .language-java}

If an application needs access to specific [`Metadata.Entry`][] objects, then it
should listen to `Player.Listener#onMetadata` (for dynamic metadata delivered
during playback). Alternatively, if there is a need to look at static metadata,
this can be accessed through the `TrackSelections#getFormat`.
`Player#getMediaMetadata` is populated from both of these sources.

## Without playback ##

If playback is not needed, it is more efficient to use the
[`MetadataRetriever`][] to extract the metadata because it avoids having to
create and prepare a player.

~~~
ListenableFuture<TrackGroupArray> trackGroupsFuture =
   MetadataRetriever.retrieveMetadata(context, mediaItem);
Futures.addCallback(
   trackGroupsFuture,
   new FutureCallback<TrackGroupArray>() {
     @Override
     public void onSuccess(TrackGroupArray trackGroups) {
       handleMetadata(trackGroups);
     }

     @Override
     public void onFailure(Throwable t) {
       handleFailure(t);
     }
   },
   executor);
~~~
{: .language-java}

## Motion photos ##

It is also possible to extract the metadata of a motion photo, containing the
image and video offset and length for example. The supported formats are:

* JPEG motion photos recorded by Google Pixel and Samsung camera apps. This
  format is playable by ExoPlayer and the associated metadata can therefore be
  retrieved with a player or using the `MetadataRetriever`.
* HEIC motion photos recorded by Google Pixel and Samsung camera apps. This
  format is currently not playable by ExoPlayer and the associated metadata
  should therefore be retrieved using the `MetadataRetriever`.

For motion photos, the `TrackGroupArray` obtained with the `MetadataRetriever`
contains a `TrackGroup` with a single `Format` enclosing a
[`MotionPhotoMetadata`][] metadata entry.

~~~
for (int i = 0; i < trackGroups.length; i++) {
 TrackGroup trackGroup = trackGroups.get(i);
 Metadata metadata = trackGroup.getFormat(0).metadata;
 if (metadata != null && metadata.length() == 1) {
   Metadata.Entry metadataEntry = metadata.get(0);
   if (metadataEntry instanceof MotionPhotoMetadata) {
     MotionPhotoMetadata motionPhotoMetadata = (MotionPhotoMetadata) metadataEntry;
     handleMotionPhotoMetadata(motionPhotoMetadata);
   }
 }
}
~~~
{: .language-java}

[documentation for the latest ExoPlayer release]: https://developer.android.com/guide/topics/media/exoplayer/retrieving-metadata
[`MediaMetadata`]: {{ site.exo_sdk }}/MediaMetadata.html
[`Metadata.Entry`]: {{ site.exo_sdk }}/metadata/Metadata.Entry.html
[`MetadataRetriever`]: {{ site.exo_sdk }}/MetadataRetriever.html
[`MotionPhotoMetadata`]: {{ site.exo_sdk }}/metadata/mp4/MotionPhotoMetadata.html
