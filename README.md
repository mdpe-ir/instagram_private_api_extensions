# Instagram Private API Extensions

An extension module to [instagram\_private\_api](https://github.com/ping/instagram_private_api) to help with common tasks such as posting a photo or video.

![](https://img.shields.io/badge/Python-2.7%2C%203.5-3776ab.svg?maxAge=2592000)
[![Release](https://img.shields.io/github/release/ping/instagram_private_api_extensions.svg?colorB=ff7043)](https://github.com/ping/instagram_private_api_extensions/releases)
[![Docs](https://img.shields.io/badge/docs-readthedocs.io-ff4980.svg?maxAge=2592000)](https://instagram-private-api-extensions.readthedocs.io/en/latest/)
[![Build](https://img.shields.io/travis/com/ping/instagram_private_api_extensions.svg)](https://travis-ci.com/ping/instagram_private_api_extensions)
[![Coverage](https://img.shields.io/coveralls/ping/instagram_private_api_extensions.svg)](https://coveralls.io/github/ping/instagram_private_api_extensions)

## Features

1. [``media``](#media): Edits a photo/video so that it complies with Instagram's requirements by:
    * Resizing
    * Cropping to fit the minimum/maximum aspect ratio
    * Generating the video thumbnail image
    * Clipping the video duration if it is too long
    * Changing the format/encoding

2. [``pagination``](#pagination): Page through an api call such as ``api.user_feed()``.

3. [``live``](#live): Download an ongoing IG live stream. Requires ffmpeg installed.

4. [``replay``](#replay): Download an IG live replay stream. Requires ffmpeg installed.

## Documentation

Documentation is available at https://instagram-private-api-extensions.readthedocs.io/en/latest/

## Install

Install with pip using

```bash
pip install git+https://git@github.com/mdpe-ir/instagram_private_api_extensions.git@0.3.9

```

To update:

```bash
pip install git+https://git@github.com/mdpe-ir/instagram_private_api_extensions.git@0.3.9 --upgrade
```

To update with latest repo code:

```bash
pip install git+https://git@github.com/ping/instagram_private_api_extensions.git --upgrade --force-reinstall
```

## Usage

### [Media](instagram_private_api_extensions/media.py)
```python
from instagram_private_api import Client, MediaRatios
from instagram_private_api_extensions import media

api = Client('username', 'password')

# post a photo
photo_data, photo_size = media.prepare_image(
    'pathto/my_photo.jpg', aspect_ratios=MediaRatios.standard)
api.post_photo(photo_data, photo_size, caption='Hello World!')

# post a video
vid_data, vid_size, vid_duration, vid_thumbnail = media.prepare_video(
    'pathto/my_video.mp4', aspect_ratios=MediaRatios.standard)
api.post_video(vid_data, vid_size, vid_duration, vid_thumbnail)

# post a photo story
photo_data, photo_size = media.prepare_image(
    'pathto/my_photo.jpg', aspect_ratios=MediaRatios.reel)
api.post_photo_story(photo_data, photo_size)

# post a video story
vid_data, vid_size, vid_duration, vid_thumbnail = media.prepare_video(
    'pathto/my_video.mp4', aspect_ratios=MediaRatios.reel)
api.post_video_story(vid_data, vid_size, vid_duration, vid_thumbnail)

# post a video without reading the whole file into memory
vid_saved_path, vid_size, vid_duration, vid_thumbnail = media.prepare_video(
    'pathto/my_video.mp4', aspect_ratios=MediaRatios.standard,
    save_path='pathto/my_saved_video.mp4', save_only=True)
# To use save_only, the file must be saved locally
# by specifying the save_path
with open(vid_saved_path, 'rb') as video_fp:
    api.post_video(video_fp, vid_size, vid_duration, vid_thumbnail)
```

### [Pagination](instagram_private_api_extensions/pagination.py)

```python
from instagram_private_api_extensions import pagination

# page through a feed
items = []
for results in pagination.page(api.user_feed, args={'user_id': '123456'}):
    if results.get('items'):
        items.extend(results['items'])
print(len(items))
```

### [Live](instagram_private_api_extensions/live.py)

```python
from instagram_private_api_extensions import live

broadcast = api.broadcast_info('1234567890')

dl = live.Downloader(
    mpd=broadcast['dash_playback_url'],
    output_dir='output_{}/'.format(broadcast['id']),
    user_agent=api.user_agent)
try:
    dl.run()
except KeyboardInterrupt:
    if not dl.is_aborted:
        dl.stop()
finally:
    # combine the downloaded files
    # Requires ffmpeg installed. If you prefer to use avconv
    # for example, omit this step and do it manually
    dl.stitch('my_video.mp4')
```

### [Replay](instagram_private_api_extensions/replay.py)

```python
from instagram_private_api_extensions import replay

user_story_feed = api.user_story_feed('12345')

broadcasts = user_story_feed.get('post_live_item', {}).get('broadcasts', [])
for broadcast in broadcasts:
    dl = replay.Downloader(
        mpd=broadcast['dash_manifest'],
        output_dir='output_{}/'.format(broadcast['id']),
        user_agent=api.user_agent)
    # download and save to file
    dl.download('output_{}.mp4'.format(broadcast['id']))
```

## Support
Make sure to review the [contributing documentation](CONTRIBUTING.md) before submitting an issue report or pull request.
