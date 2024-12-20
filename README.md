# importing necessary modules
import cv2
import numpy as np
import face_recognition
import os
from datetime import datetime
import streamlit as st

st.title("STOP CHILD-TRAFFICKING")
run = st.checkbox('RUN')
FRAME_WINDOW = st.image([])

# created a list that will get images from the folder automatically and generate encodings automatically
path = 'images_database'
images = []
classNames = []
myList = os.listdir(path)

with st.sidebar:
    st.subheader("JOIN HANDS TO")
    st.header("STOP TRAFFICKING")
    st.write("Ever seen a child lost and scared on the streets?")
    st.write("Here's a database of lost children!")
    st.subheader("POLICE : 100")

for cl in myList:
    # read images in myList
    curImg = cv2.imread(f'{path}/{cl}')
    # append image in images directory
    images.append(curImg)
    # .jpg removed from name and name stored
    classNames.append(os.path.splitext(cl)[0])


# faces are encoded so that we know which face belong to whom
def findEncodings(images):
    # empty list created
    encodeList = []
    for img in images:
        # image is converted to rgb
        img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        # finding the encodings using face_encoding function of imported face_recognition module
        encode = face_recognition.face_encodings(img)[0]
        # append encodings to encodeList
        encodeList.append(encode)
    return encodeList


# if the camera face matches one of the list face, his/her name is recorded automatically in Attendance.csv file
def markAttendance(name):
    with open('Attendance.csv', 'r+') as f:
        myDataList = f.readlines()
        nameList = []
        for line in myDataList:
            entry = line.split(',')
            nameList.append(entry[0])
        if name not in nameList:
            now = datetime.now()
            dtString = now.strftime('%H:%M:%S')
            f.writelines(f'\n{name},{dtString}')


markAttendance('a')

encodeListKnown = findEncodings(images)
print('Encoding Completed')

# laptop camera is used so id=0, if external camera need to be used id will be 1
cap = cv2.VideoCapture(0)
while run:
    # object read from camera goes to img variable
    ret, img = cap.read()
    # img is converted from bgr to rgb and stored in img
    img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    # faces img resized and stored in imgS
    imgS = cv2.resize(img, (0, 0), None, 0.25, 0.25)
    # face image is converted from bgr to rgb and stored in imgS
    imgS = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

    # find face location in current camera frame
    facesCurFrame = face_recognition.face_locations(imgS)
    # current frame encoded using face_encoding function of imported face_recognition module
    encodesCurFrame = face_recognition.face_encodings(imgS, facesCurFrame)

    for encodeFace, faceLoc in zip(encodesCurFrame, facesCurFrame):
        # matches camera image to list image
        matches = face_recognition.compare_faces(encodeListKnown, encodeFace)
        # distance between 2 images compared
        faceDis = face_recognition.face_distance(encodeListKnown, encodeFace)

        # finds minimum distance list image compared to camera image
        matchIndex = np.argmin(faceDis)

        if matches[matchIndex]:
            # name of person's face which matches the camera face is converted to upper case and stored in name variable
            name = classNames[matchIndex].upper()
            print(name)
            # rectangle created around camera face
            y1, x2, y2, x1 = faceLoc
            cv2.rectangle(img, (x1, y1), (x2, y2), (0, 255, 0), 2)
            cv2.rectangle(img, (x1, y2 - 35), (x2, y2), (0, 255, 0), cv2.FILLED)
            cv2.putText(img, name, (x1 + 6, y2 - 6), cv2.FONT_HERSHEY_COMPLEX, 0.5, (255, 255, 255), 2)
            with st.sidebar:
                st.warning(name)
                st.warning(" is VICTIM OF HUMAN TRAFFICKING, Please immediately report to police")
        else:
            # rectangle created around camera face with name UNKNOWN
            y1, x2, y2, x1 = faceLoc
            cv2.rectangle(img, (x1, y2 - 35), (x2, y2), (0, 255, 0), cv2.FILLED)
            name = 'UNKNOWN'
            cv2.putText(img, name, (x1 + 6, y2 - 6), cv2.FONT_HERSHEY_COMPLEX, 0.5, (255, 255, 255), 2)
            with st.sidebar:
                st.info("The child is not a victim of human trafficking")

        markAttendance(name)

    FRAME_WINDOW.image(img)
