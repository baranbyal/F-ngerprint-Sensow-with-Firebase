
//*******************************libraries********************************
//FIREBASE
//#include <FirebaseArduino.h>
#include <FirebaseESP8266.h>
#include <ArduinoJson.h>
//NodeMCU--------------------------
//#include <WiFiClient.h> 
#include <ESP8266WiFi.h>
#include <SoftwareSerial.h>
//#include <ESP8266HTTPClient.h>
#include <SimpleTimer.h>           //https://github.com/jfturcot/SimpleTimer
#include <Wire.h>

#include <Adafruit_Fingerprint.h>  //https://github.com/adafruit/Adafruit-Fingerprint-Sensor-Library
//************************************************************************

//asdaskjdh

// Provide the token generation process info.
#include <addons/TokenHelper.h>

// Provide the RTDB payload printing info and other helper functions.
#include <addons/RTDBHelper.h>


//Fingerprint scanner Pins
#define Finger_Rx D5 //D5
#define Finger_Tx D6 //D6


#define API_KEY "XlNzb0yQR1mejcE3qmzeJPbid2j2Dm2uGisvjb5M"
#define DATABASE_URL "fearless-five-default-rtdb.firebaseio.com"

//************************************************************************
WiFiClient client;
SimpleTimer timer;
//SimpleTimer timerLock;
SoftwareSerial mySerial(Finger_Rx, Finger_Tx);
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial);
//************************************************************************
/* Set these to your desired credentials. */



//const char *ssid = "DAMSIZ_GIRILMEZ";
//const char *password = "eeMagdurlari.2022";
const char *ssid = "WIFIKiller";


const char *password = "baranbaran";

//const char *ssid = "POCO_X3_NFC";
//const char *password = "metu3406";

//const char *ssid = "Mr. Q";
//const char *password = "19641973";
//const char *ssid = "Damash";
//const char *password = "112233456";
//const char* device_token  = "2b47dd8c61d9b985";
//const char *ssid = "TurkTelekom_ZJ99Y";
//const char *password = "4DDD3B944Ad36";
const char* device_token  = "U0T6QGQTD0G2JXGN";
//************************************************************************

int FingerID = 0, t1, t2;                           // The Fingerprint ID from the scanner 
uint8_t id;
unsigned long previousMillis = 0;


FirebaseData fbdo;

FirebaseAuth auth;
FirebaseConfig config;


bool checkCheck=true;

//************************************************************************
void setup() {

    

    pinMode(D7, OUTPUT);
    pinMode(D8, OUTPUT);

    lock();

    Serial.begin(115200);
    delay(1000);

    delay(2000); // Pause for 2 seconds
    //---------------------------------------------
    connectToWiFi();

    config.api_key = API_KEY;

    config.database_url = DATABASE_URL;
    config.signer.tokens.legacy_token = "XlNzb0yQR1mejcE3qmzeJPbid2j2Dm2uGisvjb5M";

    Firebase.begin(&config, &auth);

    Firebase.reconnectWiFi(true);

    Firebase.setDoubleDigits(5);  

    Firebase.setString(fbdo,F("FingerSensor/deviceMode/mode"),"check");

    Firebase.reconnectWiFi(true);

    Firebase.setDoubleDigits(5);

    delay(1000);

    // 
    //---------------------------------------------
    // Set the data rate for the sensor serial port
    finger.begin(57600);
    Serial.println("\n\nAdafruit finger detect test");

    if (finger.verifyPassword()) {
        Serial.println("Found fingerprint sensor!");

    } else {
        Serial.println("Did not find fingerprint sensor");

        while (1) { delay(1); }
    }

    
    Firebase.setString(fbdo,F("FingerSensor/WipeAllFingers/wipeCheck"),"false");

    //---------------------------------------------
    finger.getTemplateCount();
    Serial.print("Sensor contains "); Serial.print(finger.templateCount); Serial.println(" templates");
    Serial.println("Waiting for valid finger...");
    //Timers---------------------------------------
    timer.setInterval(25000L, CheckMode);
    t1 = timer.setInterval(10000L, ChecktoAddID);      //Set an internal timer every 10sec to check if there a new fingerprint in the website to add it.
    t2 = timer.setInterval(15000L, ChecktoDeleteID);   //Set an internal timer every 15sec to check wheater there an ID to delete in the website.
    //---------------------------------------------
    //timerLock.setInterval(25000L,checkLock);
    //---------------------------------------------
    CheckMode();
    //checkLock();
}
//************************************************************************
void loop() {

        timer.run();      //Keep the timer in the loop function in order to update the time as soon as possible

        if(checkCheck) getFingerprintID(); 
         
        delay(50);
}
//************************************************************************
// void checkLock(){

void lock(){
    digitalWrite(D7, HIGH);
    digitalWrite(D8, LOW);
}

void unlock(){
    digitalWrite(D7, LOW);
    digitalWrite(D8, HIGH);
}

void WipeFingers(){

    Firebase.setString(fbdo,F("FingerSensor/WipeAllFingers/wipeCheck"),"false");
    Firebase.setString(fbdo,F("FingerSensor/WipeAllFingers/status"),"Wiped");
    finger.emptyDatabase();
    Serial.println("Now database is empty");
}

//********************Get the Fingerprint ID******************
int getFingerprintID() {
    //Serial.println("here");
    uint8_t p = finger.getImage();
    switch (p) {
        case FINGERPRINT_OK:
        Serial.println("Image taken");
        break;
        case FINGERPRINT_NOFINGER:
        //Serial.println("No finger detected");
        return 0;
        case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        return -2;
        case FINGERPRINT_IMAGEFAIL:
        Serial.println("Imaging error");
        return -2;
        default:
        Serial.println("Unknown error");
        return -2;
    }
    // OK success!
    p = finger.image2Tz();
    switch (p) {
        case FINGERPRINT_OK:
        Serial.println("Image converted");
        break;
        case FINGERPRINT_IMAGEMESS:
        Serial.println("Image too messy");
        return -1;
        case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        return -2;
        case FINGERPRINT_FEATUREFAIL:
        Serial.println("Could not find fingerprint features");
        return -2;
        case FINGERPRINT_INVALIDIMAGE:
        Serial.println("Could not find fingerprint features");
        return -2;
        default:
        Serial.println("Unknown error");
        return -2;
    }
    // OK converted!
    p = finger.fingerFastSearch();
    if (p == FINGERPRINT_OK) {
        Serial.println("Found a print match!");
    } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
        Serial.println("Communication error");
        return -2;
    } else if (p == FINGERPRINT_NOTFOUND) {
        Serial.println("Did not find a match");
        Firebase.setString(fbdo,F("FingerSensor/Warnings"),"WrongFinger");
        return -1;
    } else {
        Serial.println("Unknown error");
        return -2;
    }   

    // found a match!
    Firebase.setString(fbdo,F("FingerSensor/Warnings"),"Recognized :) Welcome!");
    Serial.print("Found ID #"); Serial.print(finger.fingerID); 
    Serial.print(" with confidence of "); Serial.println(finger.confidence); 

    //char *output="Found ID #"+finger.fingerID+" with confidence of "+finger.confidence;

    Firebase.setInt(fbdo,F("FingerSensor/Fingers/value"),finger.fingerID);
    Firebase.setString(fbdo,F("FingerSensor/authEntry"),"true");
    Firebase.setString(fbdo,F("FingerSensor/lock"),"true");

    unlock();
    // CheckMode();

    return finger.fingerID;
    }
//******************Check if there a Fingerprint ID to delete******************
void ChecktoDeleteID(){
    Firebase.get(fbdo,F("FingerSensor/add/status"));
    String addCheck = fbdo.to<String>();
    if(addCheck!="Checking"){
        Serial.println("Check to Delete ID");
        if(Firebase.ready()){


            //FirebaseObject deleteCheck = Firebase.get("FingerSensor/delete");
            
            Firebase.get(fbdo,F("FingerSensor/delete/deleteCheck"));
            String b = fbdo.to<String>();

            if(b=="true"){
                Firebase.setString(fbdo,F("FingerSensor/delete/deleteCheck"),"false");
                Firebase.setString(fbdo,F("FingerSensor/delete/status"),"Deleting");
                Serial.println("Delete is True");

                Firebase.get(fbdo,F("FingerSensor/delete/deleteID"));

                int deleteID=fbdo.to<int>();

                deleteFingerprint(deleteID);

                delay(1000);

            }
            else{
                Serial.println("Delete is False");
            }
            
        }
    }
}
    
    //******************Delete Finpgerprint ID*****************
uint8_t deleteFingerprint( int id) {
    uint8_t p = -1;
    
    p = finger.deleteModel(id);
    
    Firebase.setInt(fbdo,F("FingerSensor/delete/lastDeleted"),id);

    if (p == FINGERPRINT_OK) {

    } else if (p == FINGERPRINT_PACKETRECIEVEERR) {

        return p;
    } else if (p == FINGERPRINT_BADLOCATION) {

        return p;
    } else if (p == FINGERPRINT_FLASHERR) {

        return p;
    } else {

        return p;
    }   

    Firebase.setString(fbdo,F("FingerSensor/delete/status"),"Done");
}

//******************Check if there a Fingerprint ID to add******************
void ChecktoAddID(){
    Firebase.get(fbdo,F("FingerSensor/add/status"));
    String addCheck = fbdo.to<String>();
    if(addCheck!="Checking"){
        Serial.println("Check to Add ID");
        if(Firebase.ready()){


            //FirebaseObject addCheck = Firebase.get("FingerSensor/add");
            
            //Serial.println(Firebase.getString(fbdo,F("FingerSensor/add/addCheck")));
            Firebase.get(fbdo,F("FingerSensor/add/addCheck"));
            String b = fbdo.to<String>();
            if(b=="true"){
                Firebase.setString(fbdo,F("FingerSensor/add/addCheck"),"false");
                Firebase.setString(fbdo,F("FingerSensor/add/status"),"Checking");
                Serial.println("Add is True");
                Firebase.get(fbdo,F("FingerSensor/add/addID"));
                int id_now = fbdo.to<int>();
                Serial.println(id_now);
                id = id_now;
                getFingerprintEnroll();

                delay(1000);

            }
            else{
                Serial.println("Add is False");
            }
        }
    }
}
//******************Check the Lock*****************
void CheckLock(){
    
}

//******************Check the Mode*****************
void CheckMode(){
    
    //if(Firebase.ready()){

        Firebase.get(fbdo,F("FingerSensor/lock"));
        String bel = fbdo.to<String>();
        if(bel=="true"){
            Firebase.setString(fbdo,F("FingerSensor/lock"),"false");
            unlock();
        }
        else if (bel=="false"){

            lock();
        }

        Serial.println("Check Mode");

        //CHECK FOR CURRENT

        Firebase.get(fbdo,F("FingerSensor/deviceMode/current"));
        String b = fbdo.to<String>();
        if(b=="check"){
            Serial.println("Check Lock");
            Firebase.get(fbdo,F("FingerSensor/authEntry"));
            String be = fbdo.to<String>();
            if(be=="true"){
                Firebase.setString(fbdo,F("FingerSensor/lock"),"true");
                Firebase.setString(fbdo,F("FingerSensor/authEntry"),"false");
            }
            else{
                Firebase.setString(fbdo,F("FingerSensor/lock"),"false");
            }
        }

        //CHECK FOR MODE
        
        //FirebaseObject deviceMode = Firebase.get("FingerSensor/deviceMode");
        Firebase.get(fbdo,F("FingerSensor/deviceMode/mode"));
        String bi = fbdo.to<String>();
        if(bi=="check"){
            Firebase.setString(fbdo,F("FingerSensor/deviceMode/current"),"check");
            Firebase.setString(fbdo,F("FingerSensor/deviceMode/mode"),"pending");
            timer.disable(t1);
            timer.disable(t2);
            Serial.println("Device Mode: Check");
            checkCheck=true;
            Firebase.get(fbdo,F("FingerSensor/lock"));
            String be = fbdo.to<String>();
            if(be=="true"){
                lock();
            }
            else if(be=="false"){
                unlock();
            }
        }
        else if(bi=="adjust"){
            Firebase.setString(fbdo,F("FingerSensor/deviceMode/current"),"adjust");
            Firebase.setString(fbdo,F("FingerSensor/deviceMode/mode"),"pending");
            timer.enable(t1);
            timer.enable(t2);
            Serial.println("Device Mode: Adjust");
            checkCheck=false;

            Firebase.get(fbdo,F("FingerSensor/WipeAllFingers/wipeCheck"));

            String wipeCheck1=fbdo.to<String>();

            if(wipeCheck1=="true"){
                WipeFingers();
            }
        }
        else{

        }

        Firebase.setString(fbdo,F("FingerSensor/Warnings"),"Neutral");
        Firebase.setString(fbdo,F("FingerSensor/WipeAllFingers/status"),"Pending");


        
    //}
    //  Serial.print("Number of Timers: ");
    //  Serial.println(timer.getNumTimers());
    // else{
    //     Serial.println("Firebase not ready");
    // }
}
//******************Enroll a Finpgerprint ID*****************
uint8_t getFingerprintEnroll() {
    int p = -1;
    // display.clearDisplay();
    // display.drawBitmap( 34, 0, FinPr_scan_bits, FinPr_scan_width, FinPr_scan_height, WHITE);
    // display.display();
    while (p != FINGERPRINT_OK) {
            
        p = finger.getImage();
        switch (p) {
        case FINGERPRINT_OK:
        //Serial.println("Image taken");

        break;
        case FINGERPRINT_NOFINGER:
        //Serial.println(".");

        break;
        case FINGERPRINT_PACKETRECIEVEERR:

        break;
        case FINGERPRINT_IMAGEFAIL:
        Serial.println("Imaging error");
        break;
        default:
        Serial.println("Unknown error");
        break;
        }
        
        
    }

    

    // OK success!
    p = finger.image2Tz(1);
    switch (p) {
        case FINGERPRINT_OK:

        break;
        case FINGERPRINT_IMAGEMESS:

        return p;
        case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        return p;
        case FINGERPRINT_FEATUREFAIL:
        Serial.println("Could not find fingerprint features");
        return p;
        case FINGERPRINT_INVALIDIMAGE:
        Serial.println("Could not find fingerprint features");
        return p;
        default:
        Serial.println("Unknown error");
        return p;
    }

    //Serial.println("Remove finger");
    delay(2000);
    p = 0;
    while (p != FINGERPRINT_NOFINGER) {
        p = finger.getImage();
    }
    Serial.print("ID "); Serial.println(id);
    p = -1;

    while (p != FINGERPRINT_OK) {
        p = finger.getImage();
        switch (p) {
        case FINGERPRINT_OK:
        //Serial.println("Image taken");

        break;
        case FINGERPRINT_NOFINGER:
        //Serial.println(".");

        break;
        case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        break;
        case FINGERPRINT_IMAGEFAIL:
        Serial.println("Imaging error");
        break;
        default:
        Serial.println("Unknown error");
        break;
        }
    }

    Firebase.setInt(fbdo,F("FingerSensor/add/lastAdded"),id);
    Firebase.setString(fbdo,F("FingerSensor/add/status"),"Done");

    // OK success!

    p = finger.image2Tz(2);
    switch (p) {
        case FINGERPRINT_OK:
        //Serial.println("Image converted");

        break;
        case FINGERPRINT_IMAGEMESS:
        //Serial.println("Image too messy");

        return p;
        case FINGERPRINT_PACKETRECIEVEERR:
        Serial.println("Communication error");
        return p;
        case FINGERPRINT_FEATUREFAIL:
        Serial.println("Could not find fingerprint features");
        return p;
        case FINGERPRINT_INVALIDIMAGE:
        Serial.println("Could not find fingerprint features");
        return p;
        default:
        Serial.println("Unknown error");
        return p;
    }
    
    // OK converted!
    Serial.print("Creating model for #");  Serial.println(id);
    
    p = finger.createModel();
    if (p == FINGERPRINT_OK) {
        Serial.println("Prints matched!");

    } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
        Serial.println("Communication error");
        return p;
    } else if (p == FINGERPRINT_ENROLLMISMATCH) {
        Serial.println("Fingerprints did not match");

        return p;
    } else {
        Serial.println("Unknown error");
        return p;
    }   
    
    Serial.print("ID "); Serial.println(id);
    p = finger.storeModel(id);
    if (p == FINGERPRINT_OK) {
        Serial.println("Stored!");

        //confirmAdding(id);
    } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
        Serial.println("Communication error");
        return p;
    } else if (p == FINGERPRINT_BADLOCATION) {
        Serial.println("Could not store in that location");
        return p;
    } else if (p == FINGERPRINT_FLASHERR) {
        Serial.println("Error writing to flash");
        return p;
    } else {
        Serial.println("Unknown error");
        return p;
    }   
}
//******************Check if there a Fingerprint ID to add******************
// void confirmAdding(int id){
//      // Serial.println("confirm Adding");

// }
//********************connect to the WiFi******************
void connectToWiFi(){
    WiFi.mode(WIFI_OFF);        //Prevents reconnection issue (taking too long to connect)
    delay(1000);
    WiFi.mode(WIFI_STA);
    Serial.print("Connecting to ");
    Serial.println(ssid);
    WiFi.begin(ssid, password);

    
    uint32_t periodToConnect = 30000L;
    for(uint32_t StartToConnect = millis(); (millis()-StartToConnect) < periodToConnect;){
      if ( WiFi.status() != WL_CONNECTED ){
        delay(500);
        Serial.print(".");
      } else{
        break;
      }
    }
    
    if(WiFi.isConnected()){
      Serial.println("");
      Serial.println("Connected");
      

      
      Serial.print("IP address: ");
      Serial.println(WiFi.localIP());  //IP address assigned to your ESP
    }
    else{
      Serial.println("");
      Serial.println("Not Connected");
      WiFi.mode(WIFI_OFF);
      delay(1000);
    }
    delay(1000);
}
//=======================================================================
