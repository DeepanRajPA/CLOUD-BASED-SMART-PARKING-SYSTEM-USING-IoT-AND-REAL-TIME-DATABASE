import cv2
import firebase_admin
from firebase_admin import credentials, db

# 🔥 Firebase setup
cred = credentials.Certificate("serviceAccountKey.json")

firebase_admin.initialize_app(cred, {
    'databaseURL': 'https://parking-system-fb7c4-default-rtdb.firebaseio.com/'
})

ref = db.reference('/parking')

cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    height, width, _ = frame.shape

    # SIMPLE LOGIC (for now)
    center_x = width // 2

    if center_x < width/3:
        slot = "A"
    elif center_x < 2*width/3:
        slot = "B"
    else:
        slot = "C"

    print("Sending:", slot)

    ref.set({
        "slot": slot
    })

    cv2.imshow("Camera", frame)

    if cv2.waitKey(1) & 0xFF == 27:
        break

cap.release()
cv2.destroyAllWindows()
