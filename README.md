# INDOOR AIR QUALITY MODELING BASED ON IoT
The purpose of this project is to provide an overview of indoor air quality (IAQ) modeling based on Internet of Things (IoT) technologies.

## Objective
The objective of this project a real-time model to monitor and control the indoor air quality.

## Abstract
In this work, a novel method is implemented using sensor fusion to make a quick and efficient response in air quality control using inlet flow control of air through a filtered channel.

## Methodology

#### Indoor Air Quality Index (IAQI) includes 

+ Integrating the pollutants.
+ Gathering the standards.
+ Selecting the monitoring Location.
+ Installing sensor fusion.
+ Calculating, Aggregating, and Interpreting IAQI.

#### Indoor Air Modeling includes

+ Collecting the data.
+ Choosing a modeling tool.
+ Building a strong model.
+ Simulating Scenarios.
+ Analysing results.
+ Implementing the improvements.

## Requirements
#### Hardware and Software Selection
1 PM2.5 GP2Y1010AU0F Dust Smoke Particle Sensor

2 MQ7 – Smoke(CarbonMonoxide)

3 Female port

4 Resister of 150ohm

5 Capacitor of 220uF

6	Breadboard

7	Single stranded wire

8 Iot enabled microcontroller node mcu wifi module

9 Arduino Software (IDE)

10 Iot platform –ThingSpeak 



## Project Architecture
![WhatsApp Image 2023-06-18 at 21 35 20](https://github.com/Gowri4622/INDOOR-AIR-QUALITY-MODELING-BASED-ON-IoT/assets/75235455/64c87a95-fb8c-4833-ab2d-2b86cc4dbfe5)

![Picture1](https://github.com/Gowri4622/INDOOR-AIR-QUALITY-MODELING-BASED-ON-IoT/assets/75235455/9a937a95-be2d-4721-889e-aef7cb253f25)

## Program        
### Hardware Program
```c
#include <ESP8266WiFi.h>
#include <SPI.h>
#include <Wire.h>
 String apiKey = "VOTEBNCMF5IJJ26L";     
const char *ssid =  "vasanths 11R 5G";     
const char *pass =  "j772xt5q";
const char* server = "api.thingspeak.com";
int measurePin = A0; 
int ledPower = 2;   
int samplingTime = 280;
int deltaTime = 40;
int sleepTime = 9680;
float voMeasured = 0;
float calcVoltage = 0;
float dustDensity = 0;
const int DOUTpin=5;
int limit;
WiFiClient client;
void setup()
{
  Serial.begin(9600);
  pinMode(ledPower,OUTPUT);
      pinMode(DOUTpin, OUTPUT);
  WiFi.begin(ssid, pass);
while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");
  }
    Serial.println("");
    Serial.println("WiFi connected");
}
int analgoread1()
{
  digitalWrite(DOUTpin,LOW);
  return analogRead(0);
}

void loop()
{
  digitalWrite(ledPower,LOW); // power on the LED
  delayMicroseconds(samplingTime);
  voMeasured = analogRead(measurePin); // read the dust value
  delayMicroseconds(deltaTime);
  digitalWrite(ledPower,HIGH); // turn the LED off
  delayMicroseconds(sleepTime);
   calcVoltage = voMeasured * (5.0 / 1024.0); 
  dustDensity = 170 * calcVoltage - 0.1;
  limit=analgoread1();
     Serial.print("dustDensity ppm"); // unit: ug/m3
   Serial.println(dustDensity); // unit: ug/m3
   Serial.print("MQ7 ppm  ");  
      Serial.println(limit); // unit: ug/m3
 if (client.connect(server, 80)) // "184.106.153.149" or api.thingspeak.com
  {
    String postStr = apiKey;
    postStr += "&field1=";
    postStr += String(dustDensity);
    postStr += "r\n";
     postStr += "&field2=";
    postStr += String(limit);
    postStr += "r\n";
    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());
    client.print("\n\n");
    client.print(postStr);
    Serial.println("Data Send to Thingspeak");
  }
    client.stop();
    Serial.println("Waiting...");
    delay(2000);      // thingspeak needs minimum
}

```

### Software Program
```python
#Function to calculate co individual pollutant index(coi)
def calculate_coi(co):
    coi=0
    if co<= 4.4:
        coi = co * 50 / 4.4
    elif co <= 9.4:
        coi = 50 + ((co - 4.4) * 50 / 5)
    elif co <= 12.4:
        coi = 100 + ((co - 9.4) * 100 / 3)
    elif co <= 15.4:
        coi = 200 + ((co - 12.4) * 100 / 3)
    else:
        coi = 300 + ((co - 15.4) * 100 / 4.6)
    
    return coi
data['coi']=data['CO'].apply(calculate_coi)
df= data[['CO','coi']]
df.head()

#Function to calculate pm2.5 individual pollutant index(pm25i)
def calculate_pm25i(pm25_concentration):
    if pm25_concentration <= 12:
        pm25i = pm25_concentration * 50 / 12
    elif pm25_concentration <= 35.4:
        pm25i = 50 + ((pm25_concentration - 12) * 50 / 23.4)
    elif pm25_concentration <= 55.4:
        pm25i = 100 + ((pm25_concentration - 35.4) * 100 / 20)
    elif pm25_concentration <= 150.4:
        pm25i = 200 + ((pm25_concentration - 55.4) * 100 / 95)
    elif pm25_concentration <= 250.4:
        pm25i = 300 + ((pm25_concentration - 150.4) * 100 / 100)
    elif pm25_concentration <= 350.4:
        pm25i = 400 + ((pm25_concentration - 250.4) * 100 / 100)
    elif pm25_concentration <= 500.4:
        pm25i = 500 + ((pm25_concentration - 350.4) * 100 / 150)
    else:
        pm25i = None  
        
    return pm25i

data['pm25i']=data['pm2_5'].apply(calculate_pm25i)
df= data[['pm2_5','pm25i']]
df.head()

def calculate_aqi(pm25i, coi, temperature, humidity):
    aqi=0
    if(pm25i>coi and pm25i>temperature and pm25i>humidity):
        aqi=pm25i
    if(coi>pm25i and coi>temperature and coi>humidity):
        aqi=coi
        
    return aqi
data['AQI']=data.apply(lambda x:calculate_aqi(x['pm25i'],x['coi'],x['Temperature'],x['Humidity']),axis=1)
df= data[['pm25i','coi','Temperature','Humidity','AQI']]
df.head()

# Calculating Air Quality Index
def calculate_aqi(pm25i, coi, temperature, humidity):
    aqi=0
    if(pm25i>coi and pm25i>temperature and pm25i>humidity):
        aqi=pm25i
    if(coi>pm25i and coi>temperature and coi>humidity):
        aqi=coi
        
    return aqi
data['AQI']=data.apply(lambda x:calculate_aqi(x['pm25i'],x['coi'],x['Temperature'],x['Humidity']),axis=1)
df= data[['pm25i','coi','Temperature','Humidity','AQI']]
df.head()

def AQI_Range(x):
    if x<=50:
        return "Good"
    elif x>50 and x<=100:
        return "Moderate"
    elif x>100 and x<=200:
        return "Poor"
    elif x>200 and x<=300:
        return "Unhealthy"
    elif x>300 and x<=400:
        return "Very Unhealthy"
    elif x>400:
        return "Hazardous"
    
df['AQI_Range']=data['AQI'].apply(AQI_Range)
df.head()

df['AQI_Range'].value_counts()

#Splitting dataset into dependent and independent column
X=df[['pm25i','coi','Temperature','Humidity']]
Y=df['AQI']
X_train,X_test,Y_train,Y_test=train_test_split(X,Y,test_size=0.2,random_state=70)
print(X_train.shape,X_test.shape,Y_train.shape,Y_test.shape)

# RandomForestRegressor
RF=RandomForestRegressor().fit(X_train,Y_train)
#predicting train
Train_preds1=RF.predict(X_train)
#predicting test
Test_preds1=RF.predict(X_test)
RMSE_train=(np.sqrt(metrics.mean_squared_error(Y_train, Train_preds1))) 
RMSE_test=(np.sqrt(metrics.mean_squared_error(Y_test, Test_preds1))) 
print("RMSE TrainingData=", str(RMSE_train)) 
print("RMSE TestData=", str(RMSE_test)) 
print('-'*50) 
print('RSquared value on train:',RF.score(X_train, Y_train)) 
print('RSquared value on test:',RF.score (X_test, Y_test))

# Logistic Regression
Train_preds2 =log_reg.predict(X_train2)
print("Model accuracy on train is: ",accuracy_score(Y_train2, Train_preds2))
Test_preds2 =log_reg.predict(X_test2)
print("Model accuracy on test is: ", accuracy_score(Y_test2,Test_preds2))
print('-'*50)
print("kappaScore is: ",metrics.cohen_kappa_score(Y_test2,Test_preds2))

# Testing with random values
log_reg.predict([[723.3,456.4,77.8,92.4]])
log_reg.predict([[200.3,46.4,77.8,92.4]])

```


## Output

### Hardware Kit
![WhatsApp_Image_2023-06-18_at_11 11 42-removebg-preview](https://github.com/Gowri4622/INDOOR-AIR-QUALITY-MODELING-BASED-ON-IoT/assets/75235455/2376cf59-b5bb-4e28-9c8d-1416ef1027f4)

### Pollutant graph
![Screenshot 2023-06-19 123329 (1)](https://github.com/Gowri4622/INDOOR-AIR-QUALITY-MODELING-BASED-ON-IoT/assets/75235455/bb28ed12-b26a-44eb-bf7b-74a543febf6d)

### AQI Calculation
![Screenshot 2023-06-18 201521](https://github.com/Gowri4622/INDOOR-AIR-QUALITY-MODELING-BASED-ON-IoT/assets/75235455/91574870-03f2-4d5a-a6b4-da870683267c)

### AQI Prediction
![Screenshot 2023-06-18 201659](https://github.com/Gowri4622/INDOOR-AIR-QUALITY-MODELING-BASED-ON-IoT/assets/75235455/620b31c8-fab1-4734-b5e4-d58887cc0f91)



## Result
The integration of sensors with IoT system was implemented and the output data has been collected and validated through the system. Thus an intelligent algorithm to model the IAQ based on industry standards was developed and the results through standard test cases are tested and validated.

## Conclusion
In conclusion, our project on predicting and modeling indoor air quality has
highlighted the importance of creating healthier indoor environments. By leveraging a comprehensive methodology that includes data collection, preprocessing, feature selection, and employing advanced modeling techniques such as LSTM, we have developed accurate IAQ prediction models.

## References
+ [Long short-term memory networks for air quality index predictio](https://ieeexplore.ieee.org/abstract/document/8489425)by S. Zheng et al. (IEEE Access, 2018).
+ [Air quality prediction using LSTM-based hybrid models](https://ieeexplore.ieee.org/abstract/document/8215599) by H. B. Zhuang et al. (IEEE International Conference on Data Mining Workshops, 2017).
+ [Prediction of PM2.5 concentration using LSTM recurrent neural networks](https://ieeexplore.ieee.org/abstract/document/8264573) by Y. Zhang et al. (IEEE International Conference on Environmental Science and Engineering, 2017).
+ [Long short-term memory network-based air quality index prediction in an internet of things environment](https://ieeexplore.ieee.org/abstract/document/8712119) by C. Li et al. (IEEE Access, 2019). 
+ [Prediction of Indoor Air Quality Using Neural Network Models](https://ieeexplore.ieee.org/abstract/document/1203299) by S. Wang et al. (IEEE Transactions on Instrumentation and Measurement, 2003). 
+ [Dynamic Indoor Air Quality Prediction Using Kalman Filter and Artificial Neural Network](https://ieeexplore.ieee.org/abstract/document/7138119) by C. Wu et al. (IEEE Transactions on Instrumentation and Measurement, 2015).






