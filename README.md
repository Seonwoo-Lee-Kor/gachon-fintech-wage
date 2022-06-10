# gachon-fintech-wage
가천대학교 금융수학과 2022-1학기 핀테크 수업 알바비를 부탁해 팀 깃허브

# 메인 코드 설명입니다.


## branch 근무시간데이터산출
###### fingerprint ##########

// 아두이노 실행코드 
// 지문 data 산출

#include <Wire.h>
#include <hd44780.h>
#include <hd44780ioClass/hd44780_I2Cexp.h>
#include <Adafruit_Fingerprint.h>
#include <LiquidCrystal_I2C.h>

#if (defined(__AVR__) || defined(ESP8266)) && !defined(__AVR_ATmega2560__)
SoftwareSerial mySerial(2, 3); 
#else
#define mySerial Serial1
#endif


hd44780_I2Cexp lcd(0x27,16,2); // lcd 객체 선언
Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial); 
String nameArr[3] = {"Ella", "Liam", "James"}; // 이름 설정
int logArr[3] = {0, 0, 0};   // log in/out을 위한 설정

extern volatile unsigned long timer0_millis;
unsigned long start = 0;  // 시작 시간
unsigned long end = 0;    // 종료 시간

void setup()
{
  // 시리얼 통신 시작
  Serial.begin(9600);
  // lcd 통신 시작
  lcd.begin(16, 2);

  // 출력될 라벨(타이틀) 설정
  Serial.println("ID, NAME, LOG, TIME");
  
  while (!Serial);  
  delay(100);
  //Serial.println("\n\nAdafruit finger detect test");

  // set the data rate for the sensor serial port
  // 지문인식 센서 통신 시작
  finger.begin(57600);
  delay(5);
  //지문인식 센서 디바이스 연결 확인
  if (finger.verifyPassword()) {
    //Serial.println("Found fingerprint sensor!");
  } else {
    //Serial.println("Did not find fingerprint sensor :(");
    while (1) { delay(1); }
  }


  //등록된 지문 유무 판단
  finger.getTemplateCount();

  if (finger.templateCount == 0) {
    //Serial.print("Sensor doesn't contain any fingerprint data.");
  }
  else {
//    Serial.println("Waiting for valid finger...");
//      Serial.print("Sensor contains "); Serial.print(finger.templateCount); Serial.println(" templates");
  }
}

void loop()                     // run over and over again
{
  getFingerprintID();   
  delay(50);            
}
//lcd 설정
uint8_t getFingerprintID() {
  uint8_t p = finger.getImage();  
  lcd.clear();   
  lcd.setCursor(0,0);
  lcd.print("Put your finger"); // 아두이노가 연결되었는지 확인하기 위해 초기 lcd 화면 입니다.
  lcd.setCursor(0, 1);
  lcd.print("on the sensor!");
  delay(3000);

  // 오류시 발생 lcd 화면
  switch (p) {
    case FINGERPRINT_OK:
      //Serial.println("Image taken");
      break;
    case FINGERPRINT_NOFINGER:
      //Serial.println("No finger detected");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      //Serial.println("Communication error");
      return p;
    case FINGERPRINT_IMAGEFAIL:
      //Serial.println("Imaging error");
      return p;
    default:
      //Serial.println("Unknown error");
      return p;
  }

  // OK success!

// 오류시 발생 lcd 화면
  p = finger.image2Tz();
  switch (p) {
    case FINGERPRINT_OK:
      //Serial.println("Image converted");
      break;
    case FINGERPRINT_IMAGEMESS:
      //Serial.println("Image too messy");
      return p;
    case FINGERPRINT_PACKETRECIEVEERR:
      //Serial.println("Communication error");
      return p;
    case FINGERPRINT_FEATUREFAIL:
      //Serial.println("Could not find fingerprint features");
      return p;
    case FINGERPRINT_INVALIDIMAGE:
      //Serial.println("Could not find fingerprint features");
      return p;
    default:
      //Serial.println("Unknown error");
      return p;
  }

  // OK converted!
  p = finger.fingerSearch();
  if (p == FINGERPRINT_OK) {
    //Serial.println("Found a print match!");
  } else if (p == FINGERPRINT_PACKETRECIEVEERR) {
    //Serial.println("Communication error");
    return p;
  } else if (p == FINGERPRINT_NOTFOUND) {
    //Serial.println("Did not find a match");
    return p;
  } else {
    //Serial.println("Unknown error");
    return p;
  }


 // found a match!
// 지문인식 성공 시 lcd화면 출력설정
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("#");
  lcd.print(finger.fingerID);  // ID
  lcd.print(" ");
  lcd.print(nameArr[finger.fingerID - 1]); // 이름
  lcd.print(" ");
  
// 지문인식 성공 시 시리얼 모니터 출력
  Serial.print(finger.fingerID);  
  Serial.print(", ");
  Serial.print(nameArr[finger.fingerID - 1]);
  Serial.print(", ");

// log in/out 설정
  if(logArr[finger.fingerID - 1] == 0){
    logArr[finger.fingerID - 1] = 1; // Log In 했다는 것을 남기기 위해 In하면 1로 바꿔줍니다.
    Serial.print("Log In");
    timer0_millis=0; // millis()함수를 쓰기 위해 작동시간을 초기화합니다.
    Serial.print(", ");
    Serial.println(start);
    lcd.print("Log In");
    lcd.setCursor(0, 1);
    lcd.print("Have a nice day!"); // 처음 찍을 때 lcd화면 출력설정입니다. ID, 이름, Log In, 0 순으로 출력됩니다. 
  }
  else{
    logArr[finger.fingerID - 1] = 0; // 1이라면 Log In 했다는 의미이고 그것을 다시 0으로 설정해주므로써 Out을 의미합니다.
    Serial.print("Log Out");
    Serial.print(", ");
    Serial.println(millis()); #
    timer0_millis=0; // 다시 작동시간을 0으로 설정하므로써 시간오류가 발생하는 것을 방지해줍니다.
    lcd.print("Log Out");
    lcd.setCursor(0, 1);
    lcd.print("See you again!"); // 두번째 찍을 때 lcd화면 출력설정입니다. ID, 이름, Log Out, 시간 순으로 출력됩니다.
  }
  delay(2000); // 여러번 지문이 찍히는 것을 방지하기 위해 delay 시간을 이용하여
               // 전체적인 아두이노가 오류 없이 진행되도록 합니다. 
                


  return finger.fingerID;
}







###### data_main ##########
# 아두이노 데이터 산출
import serial   
import pandas as pd
ard=serial.Serial('COM5',9600) #아두니오와 파이썬 연결
s = ""
ard.inWaiting()  # 현재 아두이노에서 불러올 수 있는 값 확인

while ard.inWaiting() > 0 :
    s += ard.readline().decode()

print(s)   # 산출된 데이터

    # 시간은 밀리초로 나오기때문에 0.000초로 나옵니다.

ard.inWaiting() #대기 중인 값 재확인
w=s.split(" ") #산출된 데이터 가공작업
count = w.count('Out,') # Out을 기준으로 하기 위해 먼저 Out한 개수 파악

# 3명의 데이터
p1=list()
p2=list()
p3=list()



for x in range(count) : 
    if float(w.index('Out,')) > 0 : # Out한 데이터가 있다면
        if w[w.index('Out,')-3][len(w[w.index('Out,')-3])-2:len(w[w.index('Out,')-3])-1]   == '1':  # ID가 1일때
            if len(w[w.index('Out,')+1]) >7 : # 시간자리가 7개 초과일때 (이유는 시간데이터 뒤에 \r\n 띄어쓰기도 포함되어 있습니다.)
                p1 += [w[w.index('Out,')+1][:-7]]   # Out한 시간데이터를 p1이라는 변수에 모으는 과정
                w.remove('Out,')                    # 모은 데이터를 제거하여 중복 방지 
            else :
                p1 += [w[w.index('Out,')+1][:-5]]   # 시간자리가 5개 초과일때(마지막에 Out한 시간데이터는 띄어쓰기가 없기 때문입니다.)
                w.remove('Out,')
      
        elif w[w.index('Out,')-3][len(w[w.index('Out,')-3])-2:len(w[w.index('Out,')-3])-1]    == '2':  # ID가 2일때
            if len(w[w.index('Out,')+1]) >7 :     
                p2 += [w[w.index('Out,')+1][:-7] ]
                w.remove('Out,')
            else :
                p2 += [w[w.index('Out,')+1][:-5]]
                w.remove('Out,')
      
        elif w[w.index('Out,')-3][len(w[w.index('Out,')-3])-2:len(w[w.index('Out,')-3])-1]    == '3':  # ID가 3일때
            if len(w[w.index('Out,')+1]) >7 :
                p3 += [w[w.index('Out,')+1][:-7]] 
                w.remove('Out,')
            else :
                p3 += [w[w.index('Out,')+1][:-5]]
                w.remove('Out,')
        



 # ID 1 인 사람 시간 data = p1
 # ID 2 인 사람 시간 data = p2
 # ID 3 인 사람 시간 data = p3

# 가정
# 현재 저희 코딩에서는 한달에 6번 일하도록 되어 있습니다.
# 이유는 한달이라는 데이터를 모으기 위해서는 한달이라는 시간이 필요했고 
# 저희는 계산식이 제대로 되는지 보여주기 위해 40시간 이상인경우 , 15시간이상 ~ 40시간미만인 경우, 15시간 미만인 경우가 필요합니다.
# 하여 초단위로 보여드리기 위해서는 6번이라는 한정된 횟수로 할 수 밖에 없었습니다.

# 주 5일 일한다면 다음과 같이 데이터를 산출 할수 있습니다.
# work_time = pd.DataFrame([p[0],p[1],p[2],p[3],p[4],0,0],
#                          columns=["1st"],index=['mon','tue','wen','thu','fri','sat','sun'])
#
#    work_time['2nd']=(p[5],p[6],p[7],p[8],p[9],0,0)
#    work_time['3rd']=(p[10],p[11],p[12],p[13],p[14],0,0)
#    work_time['4th']=(p[15],p[16],p[17],p[18],p[19],0,0)
#    work_time['H']=(9500)
#    work_time
 
 
# 개인별로 모은 시간데이터를 이용하여 데이터프레임으로 변환하는 작업코드입니다.
## 핵심은 Out한 시간을 기준으로 가동되고 아직 Out하지 않았다면 0으로 산출됩니다.

# ID 1
for x in  range(6- len(p1)) :  # 6번미만으로 일했을 경우 나머지는 0으로 채워집니다.
    if 6 - len(p1) > 0 :
        p1 += ['0']
        
try :
    work_time1 = pd.DataFrame([p1[0],0,0,0,0,0,0], 
                          columns=["1st"],index=['mon','tue','wen','thu','fri','sat','sun'])

    work_time1['2nd']=(0,0,p1[1],0,0,0,0)
    work_time1['3rd']=(0,0,0,p1[2],0,p1[3],0)
    work_time1['4th']=(p1[4],0,0,p1[5],0,0,0)
    work_time1['H']=(9500)
    work_time1 # ID 1 인 사람의 한달 데이터

except:
        print("check Id 1 data")   

# ID 2
for x in  range(6- len(p2)) :
    if 6 - len(p2) > 0 :
        p2 += ['0']
        
try :
    work_time2 = pd.DataFrame([0,0,0,p2[0],0,0,0],
                          columns=["1st"],index=['mon','tue','wen','thu','fri','sat','sun'])

    work_time2['2nd']=(0,0,p2[1],0,0,p2[2],0)
    work_time2['3rd']=(0,0,0,p2[3],0,p2[4],0)
    work_time2['4th']=(0,0,0,p2[5],0,0,0)
    work_time2['H']=(9500)
    work_time2 # ID 2 인 사람의 한달 데이터

except:
        print("check Id 2 data")
    
# ID 3
for x in  range(6- len(p3)) :
    if 6 - len(p3) > 0 :
        p3 += ['0']
        
try :
    work_time3 = pd.DataFrame([0,p3[0],0,0,p3[1],0,0],
                          columns=["1st"],index=['mon','tue','wen','thu','fri','sat','sun'])

    work_time3['2nd']=(0,0,p3[2],0,0,0,0)
    work_time3['3rd']=(0,0,0,p3[3],p3[4],0,0)
    work_time3['4th']=(0,0,0,p3[5],0,0,0)
    work_time3['H']=(9500)
    work_time3 # ID 3 인 사람의 한달 데이터

except:
        print("check Id 3 data")

# 데이트프레임 저장
work_time1.to_pickle("work_time1.pkl")
work_time2.to_pickle("work_time2.pkl")
work_time3.to_pickle("work_time3.pkl")





## branch 데이터를통해월근무시간과월급액산출



## branch 예정지급월급액과지갑에있는금액비교




