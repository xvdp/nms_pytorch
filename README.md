# torchvision_extra
torchvision functions and not included in torchvision, including
standalone version of nms included with https://github.com/facebookresearch/maskrcnn-benchmark

I needed non maximum suppression outside the scope of maskrcnn so I extracted the nms portion of the code and exposed the functions:

```
.nms(boxes, scores, shape, threshold, mode) 
```
new function, wraps prepare_boxlist() and boxlist_nms()
TODO expand functionality
```
._nms() # this is the _C function : returns a mask tensor 
.prepare_boxlist()
.boxlist_nms()

.remove_small_boxes()
.boxlist_iou()
```
### Changes to original code
* 0.1.0; extended nms(): returns tuple of keptboxes, keptscores, keepindices: this is a departure from original code source and C nms, which returns ByteTensor mask
* 0.0.9; added warpper function nms(), renamed original nms to _nms()
* 0.0.8; renamed to torchvision_extra
* 0.0.7; nms_pytorch C extensions are now installed instead of JIT loaded

## Installation
Similar to benchmark_maskrcnn - minor differences noted here. Testing WIP.
```
# in your conda environment
# ...
conda install ninja yacs cython matplotlib 
conda install cudatoolkit=9 (or =10) ensure cuda tooklit matches your distribution 

# install pytorch and torchvision 
# maskrcnn_benchmark calls for own compilation of vision and for pytorch nightly
# this works with either one
conda install pytorch torchvision -c pytorch

git clone https://github.com/xvdp/torchvision_extra
cd nms_pytorch
python setup.py build develop
```
## Use instructions / Examples

.prepare_boxlist()
Args:
    boxes:          torch.Size([N, 4]) of predicted boxes
    scores:         torch.Size([N, 1]) or torch.Size([N]) of predicted scores
    image_shape:    tuple as generated by ndarray(PilImage) or cv2.imread
    mode:           str, "xyxy", "yxyx", "xywh" or "yxhw"

```
#python
import torchvision_extra
boxlist = torchvision_extra.prepare_boxlist(boxes, scores, image.shape, mode="xyxy")
Out[*]: BoxList(num_boxes=798, image_width=399, image_height=899, mode=xyxy)

filtered = torchvision_extra.boxlist_nms(boxlist, 0.5)
Out[*]: BoxList(num_boxes=262, image_width=399, image_height=899, mode=xyxy)

# filtered.bbox # unsuppressed boxes
# filtered.extra_fields["scores"] # score tensor

# Example
boxesnms, scoresnms, keep = nms(bboxes, scores, shape, mode="xyxy")
#    Arguments:
#	bboxes:	torch.Tensor of size (N,4) dtype torch.float32 or torch.float64 : has to match scores
#	scores:	torch.Tensor of size (N,1) or (N) dtype torch.float32 or torch.float64
#	shape: 	tuple or tensor.Size: image shape
#	mode:	str: xyxy, xywh, yxyx, yxhw
#    Returns:
#	tuple: boxesnms, scoresnms, keep
#	    boxesnms:  torch.Tensor of size (M,4) of same dtype as input bboxes, where M <= N, suppressed boxes
#	    scoresnms: torch.Tensor of size (M,1)
#	    keep:      torch.ByteTensor size (M)
```
