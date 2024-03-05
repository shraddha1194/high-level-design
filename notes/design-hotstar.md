# Design Hotstar

### Live-streaming content
What is the difference between live and regular content?
 
We don't have enough time for ingestion and processing. 

There is a place where an event is happening, the live content is getting recorded at that place.
This live feed is getting translated to a production center (RTMP).

The video stream is getting translated.

There is someone who selects which screen needs to be selected and displayed at a time on feed.

The real problem is how to send this live content to millions of users.

**Is there any latency involved in this?**

Answer is Yes.

In live streaming the chunk size should be way smaller, let's say 15-30 seconds each.