# Automated-Loading-carton-counter-by-Yolo-Model
 Line-Crossing Carton Counter (Module 2)

This repository/module implements an automated computer vision pipeline designed to count cardboard shipping cartons as they are loaded from a factory/warehouse into a delivery truck. 

## 📌 Goal & Overview
The system uses a fine-tuned object detector (YOLOv8) paired with a multi-object tracker (ByteTrack) to track and count cartons uniquely and directionally as they cross a user-defined virtual line.

---

## ⚙️ Key Features & Constraints

* **Directional Counting**: Only tracks and counts cartons moving in the **Factory -> Truck** direction (e.g., Left-to-Right). Reverse movements (Truck -> Factory) are ignored.
* **No Double Counting**: Each carton is tracked using a unique `track_id` assigned by ByteTrack, ensuring it is counted only once even if it hovers or moves back and forth near the line.
* **Independent Multi-Object Tracking**: Multiple separate cartons carried by different laborers crossing the line simultaneously are counted independently.
* **Operational Assumption**: Cartons crossing the virtual line are not physically stacked (they are carried individually or side-by-side).

---

## 🛠️ Pipeline Workflow

### 1. Environment & Dependencies Setup
Install required libraries including Ultralytics (for YOLO) and headless OpenCV:
bash
pip install ultralytics opencv-python-headless
2. Assets Upload
Upload your custom fine-tuned YOLO model (best.pt).
Upload a test video representing the loading area view.
3. Coordinate Definition (Virtual Line)
Using the first frame preview from the input video, determine the pixel coordinates for the virtual line and define the "factory" side where the movement originates:

LINE_START = (600, 0)      # (x1, y1)
LINE_END = (600, 720)      # (x2, y2)
FACTORY_SIDE = 'left'      # 'left', 'right', 'top', or 'bottom'
4. Line-Crossing Algebraic Logic
The system uses the vector cross-product of the line segment and the tracked object's centroid to instantly classify which side of the line a carton is currently on: $$\text{Side Sign} = (x_2 - x_1) * (y - y_1) - (y_2 - y_1) * (x - x_1)$$

A state tracker (track_state) stores the historical side of each active ID.
A crossing event is registered when the sign transitions from Factory Side to Truck Side.
5. Tracking & Count Execution
Streams inference on the video to minimize memory overhead:

Uses standard bytetrack.yaml configuration with a confidence threshold of 0.3.
Automatically draws green bounding boxes for counted cartons, and orange for uncounted ones.
Overlays a real-time counter and the virtual line on the processed video frames, saving the results to counted_output.mp4.
📊 Evaluation & Verification Checklist
When validating the output counted_output.mp4, verify:

Does the final printed count match your manual tally of cartons crossing the line?
Are cartons that remain on the factory side without crossing properly ignored?
Is reverse movement (e.g. walking a carton back inside) ignored?
Are multiple concurrent crossings correctly tracked and separately counted?
