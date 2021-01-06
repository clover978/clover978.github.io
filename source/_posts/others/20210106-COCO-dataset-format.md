---
title: COCO dataset format
date: 2021-01-06 19:28:43
tags: 
  - COCO
categories:
  - others
---

```python
- annotations
    - 'info': dict
    - 'licenses' list
    - 'images'
    [
        {
        - 'filename': str
        - 'id': int
        - 'height': int
        - 'width': int
        - 'flickr_url': str
        - 'license': int
        - 'coco_url': str
        - 'date_captured': str
        }, 
        ...
        ...
    ]
    - 'annotations'
    [
        {
        - 'segmentation': list
        - 'num_keypoints': int
        - 'area': float
        - 'is_crowd': bool
        - 'keypoint': 
            - x1, y1, k1   # k=0 不可见未标注；k=1 不可见标注； k=2 可见标注
            - x2, y2, k2
            - ......
        - 'bbox': list
            - ( x1, y1, w1, h1 )
            - ( x2, y2, w2, h2 )
            - ......
        - 'category_id': int
        - 'image_id': int
        - 'id': int
        }, 
        ...
        ...
    ]
    - 'categories':
    [
        {
        - 'id': int
        - 'keypoints':
            - kpt-name1
            - kpt-name2
            - ...
        - 'skeleton': 
            - (pair1_index1, pair1_index2)
            - (pair2_index1, pair2_index2)
            - ...
        - 'name': 'person'
        - 'supercategory': 'person'
        }
    ]
```