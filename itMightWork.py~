import cv
import cv2
import serial
import time
import sys
import math
import plotly.plotly as py
from plotly.graph_objs import *


class Target:

    def __init__(self):
        self.capture = cv.CaptureFromCAM(1) # I want it to be  "198.168.0.6:8020.mjpeg" (0) for onboard cam 2 for mobouis 
        cv.NamedWindow("Target", 1)

    def run(self):
        # Capture first frame to get size

        frame = cv.QueryFrame(self.capture)
        frame_size = cv.GetSize(frame)
        grey_image = cv.CreateImage(cv.GetSize(frame), cv.IPL_DEPTH_8U, 1)
        moving_average = cv.CreateImage(cv.GetSize(frame), cv.IPL_DEPTH_32F, 3)
        difference = None
        ser = serial.Serial("/dev/ttyACM0", 9600, timeout = 1) 
        connected = 0 
        while connected == 0: 
            print("not connected") #Just debug message
            ser.write('c') #Seeing if the arduino is open to com yet
            if ser.read() == '4': #Arduino saying it is open to com
                 connected = 1 
                 print("connected")
                 ser = serial.Serial("/dev/ttyACM0", 9600, timeout = .1)
            time.sleep(1)

            print("it has slept")
        timesAround = []
        TimesAround = 0
        xSpeed = 0
        ySpeed = 0
        xCompare = 0
        yCompare = 0
        xVector = 0
        yVector = 0
        moved = False
        xPlot = []
        yPlot = []
        while connected == 1:
            print("it worked")
            # Capture frame from webcam
            color_image = cv.QueryFrame(self.capture)

            # Smooth to get rid of false positives
            cv.Smooth(color_image, color_image, cv.CV_GAUSSIAN, 3, 0)

            if not difference:
                # Initialize
                difference = cv.CloneImage(color_image)
                temp = cv.CloneImage(color_image)
                cv.ConvertScale(color_image, moving_average, 1.0, 0.0)
            else:
                cv.RunningAvg(color_image, moving_average, 0.020, None)

            # Convert the scale of the moving average.
            cv.ConvertScale(moving_average, temp, 1.0, 0.0)

            # Minus the current frame from the moving average.
            cv.AbsDiff(color_image, temp, difference)

            # Convert the image to grayscale.
            cv.CvtColor(difference, grey_image, cv.CV_RGB2GRAY)

            # Convert the image to black and white.
            cv.Threshold(grey_image, grey_image, 70, 255, cv.CV_THRESH_BINARY)

            # Dilate and erode to get object blobs
            cv.Dilate(grey_image, grey_image, None, 18)
            cv.Erode(grey_image, grey_image, None, 10)

            # Calculate movements
            storage = cv.CreateMemStorage(0)
            contour = cv.FindContours(grey_image, storage, cv.CV_RETR_CCOMP, cv.CV_CHAIN_APPROX_SIMPLE)
            points = []

            while contour:
                # Draw rectangles
                bound_rect = cv.BoundingRect(list(contour))
                contour = contour.h_next()

                pt1 = (bound_rect[0], bound_rect[1])
                pt2 = (bound_rect[0] + bound_rect[2], bound_rect[1] + bound_rect[3])
                points.append(pt1)
                points.append(pt2)
                cv.Rectangle(color_image, pt1, pt2, cv.CV_RGB(255,0,0), 1)

            num_points = len(points)
            if num_points:
                # Draw bullseye in midpoint of all movements
                x = y = 0
                for point in points:
                    x += point[0]
                    y += point[1]
                x /= num_points
                y /= num_points
                center_point = (x, y)
                cv.Circle(color_image, center_point, 40, cv.CV_RGB(255, 255, 255), 1)
                cv.Circle(color_image, center_point, 30, cv.CV_RGB(255, 100, 0), 1)
                cv.Circle(color_image, center_point, 20, cv.CV_RGB(255, 255, 255), 1)
                cv.Circle(color_image, center_point, 10, cv.CV_RGB(255, 100, 0), 5)
                
                TimesAround = TimesAround + 1.0 
                # Increases the amount of frames looked at 
                timesAround.append(TimesAround)
             
                xPlot.append(center_point[1])
                # The x value list for plotly
    
                yPlot.append(center_point[0])
                # The y value list for plotly              
                


                ySpeed = float((abs(center_point[1]) - abs(yCompare))) #/ timesAround
                xSpeed = float((abs(center_point[0]) - abs(xCompare))) #/ timesAround
                print(xSpeed) 
                # distance travelled sense last check

                if center_point[0]> xCompare:
                    xVector = -(xSpeed)
                elif center_point[0] < xCompare:
                    xvector = xSpeed 
                if center_point[1] > yCompare:
                    yVector = -(ySpeed)
                elif center_point[1] < yCompare:
                    yVector = ySpeed 
                  # Just making sure math isn't backwards
                   
                xProjected = center_point[0] + xSpeed
                yProjected = center_point[1] + ySpeed
                # Getting the position to check for movement from 

                
                    
                xCompare = center_point[0]  
                yCompare = center_point[1] 
                # Re-Writing the points for comparision the next loop around       
                    
                   
                    
                if math.sqrt(abs(xSpeed*xSpeed + ySpeed*ySpeed)) > 10: 
                # If the movement was below 
                    moved = True

                    #############serial communication && error smoothing###############
                if  abs(xProjected - center_point[0]) < 1 * abs(xVector) or abs(yProjected - center_point[1]) < 1 * abs(yVector) and moved == True:
                    # This control assumes that every time acceration is over 2x what it was before it is an outlier and doesn't count
                    
                    

                    read = ser.read()
                    # Reading Serial 

                    # The data being sent
                    ser.write(str(center_point[0]))
                    # First Number (X cord.)
                    ser.write("|")
                    # Data Break
                    ser.write(str(center_point[1]))
                    # Second Number (Y cord.)
                    ser.write("|") # Data Break
                    #time.sleep(.5)
                    


                    print(center_point[0])
                    print(ser.read(3))
                    ser.flushInput()
                    ser.flushOutput()
              
                elif abs(xProjected - center_point[0]) > 1 * abs(xVector) or abs(yProjected - center_point[1]) > 1 * abs(yVector):
                    print("too Fast")
                    

            # Display frame to user
            cv.ShowImage("Target", color_image)

            # Listen for ESC or ENTER key
            c = cv.WaitKey(7) % 0x100
            if c == 27 or c == 10: 
                ser.close
                trace1 = Scatter(
                    x = timesAround, 
                    y = xPlot, 
                    mode = 'markers'
                )
                trace2 = Scatter(
                    y = yPlot,
                    x = timesAround,
                    mode = 'markers'
                )
                data = Data([trace1, trace2])
                plot_url = py.plot(data, filename='basic-line')
                break
                
if __name__=="__main__":
    t = Target()
    t.run()
    
############################################################################################################
## Boolean variable that will represent 
## whether or not the arduino is connected
#connected = False

## open the serial port that your ardiono 
## is connected to.
#ser = serial.Serial("COM11", 9600)

## loop until the arduino tells us it is ready
#while not connected:
#    serin = ser.read()
 #   connected = True

## Tell the arduino to blink!
#ser.write("1")

## Wait until the arduino tells us it 
## is finished blinking
#while ser.read() == '1':
#    ser.read()

## close the port and end the program
#ser.close()
###############################################################################################################

