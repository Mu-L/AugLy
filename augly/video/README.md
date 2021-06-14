# Video

## Installation Notes

In order to run the video tests and/or use the augmentations, please install ffmpeg. If you're using conda you can do this with:
```bash
conda install -c conda-forge ffmpeg
```

You also have to make sure your ffmpeg & ffprobe binaries are available in the right locations:
```bash
which ffmpeg
sudo ln -s <ffmpeg_path> /usr/local/bin/ffmpeg
which ffprobe
sudo ln -s <ffprobe_path> /usr/local/bin/ffprobe
```

## Augmentations

For a full list of available augmentations, see [here](augly/video/__init__.py).

Our video augmentations use FFMPEG & OpenCV as the backend. All functions accept a path to the video to be augmented as input and output a file containing the augmented video. If no output path is specified, the original video input path will be overwritten. The file path to which the augmented video was written will also be returned by all augmentations.

### Function-based

You can call the functional augmentations like so:
```python
import aml.augly.video as vidaugs

video_path = "your_vid_path.mp4"
output_path = "your_output_path.mp4"

vidaugs.add_dog_filter(video_path, output_path)
vidaugs.rotate(output_path, degrees=30)   # output_path will be overwritten
```

### Class-based

We have also defined class-based versions of all augmentations as well. We have also added special Compose and ToTensor operators to facilitate using these augmentations with PyTorch:
```python
import aml.augly.video as vidaugs
from aml.augly.video import torchaugs

COLOR_JITTER_PARAMS = {
    "brightness_factor": 0.15,
    "contrast_factor": 1.3,
    "saturation_factor": 2.0,
}

AUGMENTATIONS = [
    vidaugs.ColorJitter(**COLOR_JITTER_PARAMS),
    vidaugs.HorizontalFlip(),
    vidaugs.OneOf(
        [
            vidaugs.RandomEmojiOverlay(),
            vidaugs.RandomIGFilter(),
            vidaugs.Shift(x_factor=0.25, y_factor=0.25),
        ]
    ),
]

TRANSFORMS = vidaugs.Compose(AUGMENTATIONS)
TENSOR_TRANSFORMS = torchaugs.Compose(AUGMENTATIONS + [torchaugs.ToTensor()])

video_path = "your_vid_path.mp4"
out_video_path = "your_output_path.mp4"

TRANSFORMS(video_path, out_video_path)  # transformed video now stored in `out_video_path`
video_tensor, audio_tensor, info = TENSOR_TRANSFORMS(video_path)
```

## Unit Tests

You can run our video unit tests by cloning `augly` (see [here](augly/README.md)) and then running the following:
```bash
python -m unittest augly.tests.video_tests.functional.composite_tests
python -m unittest augly.tests.video_tests.functional.cv2_tests
python -m unittest augly.tests.video_tests.functional.ffmpeg_tests
python -m unittest augly.tests.video_tests.functional.image_based_tests

python -m unittest augly.tests.video_tests.functional.composite_tests
python -m unittest augly.tests.video_tests.functional.cv2_tests
python -m unittest augly.tests.video_tests.functional.ffmpeg_tests
python -m unittest augly.tests.video_tests.functional.image_based_tests
```

Note: Some of the unit tests may fail depending which specific versions of some libraries you are running, because the behavior of some functions is slightly different and causes slightly different output video files.