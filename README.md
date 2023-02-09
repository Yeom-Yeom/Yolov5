# Yolov5

# git clone Yolov5
!git clone https://github.com/ultralytics/yolov5
%cd yolov5
!pip install -r requirements.txt

# install pytorch
!pip install torch==1.7.1+cu110 torchvision==0.8.2+cu110 torchaudio==0.7.2 -f https://download.pytorch.org/whl/torch_stable.html

#install roboflow
!pip install -q roboflow
from roboflow import Roboflow
rf = Roboflow (model_format="yolov5", notebook="roboflow-yolov5")

# get roboflow data
%cd /content
!curl -L "https://public.roboflow.com/ds/K9IjAYLk5y?key=9th6uXdcDR"> roboflow.zip; unzip roboflow.zip; rm roboflow.zip

# check data
%cat data.yaml

# make custom data
import yaml
with open("data.yaml",'r') as stream:
  num_classes = str(yaml.safe_load(stream)['nc'])
 
from IPython.core.magic import register_line_cell_magic
@register_line_cell_magic
def writetemplate (line, cell):
  with open(line,'w') as f:
    f.write(cell.format(**globals()))
    
# train
%%time
%cd /content/yolov5
!python train.py --img 3200 --batch 32 --epochs 100 --data '../data.yaml' --cfg ./models/custom_yolov5m.yaml --weights '' --name yolov5s_results --cache

# check validation
%cd /content/yolov5/
!python val.py --img 3200 --data '../data.yaml' --weights /content/yolov5/runs/train/yolov5s_results/weights/best.pt

# detect object
%cd /content/yolov5/
!python detect.py --weights /content/yolov5/runs/train/yolov5s_results/weights/best.pt --img 3200 --conf 0.4 --source /content/yolov5/test_1/images --save-crop

# save as zip
!zip -r /content/yolov5/exp.zip /content/yolov5/runs/detect/exp3
