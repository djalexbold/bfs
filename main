#include <LiquidCrystal_I2C.h> // LCD дисплей 4x20, работает по шине I2C
#include <Wire.h>              
#include <Rotary.h> 
#include <si5351.h>
#include <Keypad.h> // матричная клавиатура

#define IF  10700                // Enter your IF frequency, ex: 455 = 455kHz, 10700 = 10.7MHz, 0 = to direct convert receiver or RF generator, + will add and - will subtract IF offfset.
#define FREQ_INIT    100000000   // Enter your initial frequency at startup, ex: 7000000 = 7MHz, 10000000 = 10MHz, 840000 = 840kHz.
#define XT_CAL_F     33000     // Si5351 calibration factor, adjust to get exatcly 10MHz. Increasing this value will decreases the frequency and vice versa.
#define tunestep     10        // Change the pin used by encoder push button if you want.
#define ENCODER_A    2         // Encoder pin A 
#define ENCODER_B    3         // Encoder pin B 
#define sMetr        A6        // S-metr input pin ananlog 

unsigned long freq = FREQ_INIT;
unsigned long freqold, fstep;
unsigned long long pll_freq = 40000000000ULL;
long interfreq = IF;
long cal = XT_CAL_F;
unsigned int period = 100;   //millis display active
unsigned long time_now = 0;  //millis display active
byte encoder = 1;
byte stp;

const byte ROWS = 4; //four rows
const byte COLS = 3; //three columns
char keys[ROWS][COLS] = {
  {'1','2','3'},
  {'4','5','6'},
  {'7','8','9'},
  {'*','0','#'}
};

byte rowPins[ROWS] = {A0, A1, A2, A3}; //connect to the row pinouts of the keypad
byte colPins[COLS] = {4, 5, 6}; //connect to the column pinouts of the keypad

String input;
int counter;
const int maxLength = 5;


LiquidCrystal_I2C lcd(0x3f, 20, 4);
Rotary r = Rotary(ENCODER_A, ENCODER_B);
Si5351 si5351;
Keypad keypad = Keypad( makeKeymap(keys), rowPins, colPins, ROWS, COLS );

ISR(PCINT2_vect) {
  unsigned char result = r.process();
  if (result == DIR_CW) set_frequency(1);
  else if (result == DIR_CCW) set_frequency(-1);
}

void set_frequency(short dir) {
  if (encoder == 1) {                         //Up/Down frequency limit
    if (dir == 1) freq = freq + fstep;
    if (freq >= 160000000) freq = 160000000;
    if (dir == -1) freq = freq - fstep;
    if (fstep == 1000000 && freq <= 1000000) freq = 1000000;
    else if (freq < 10000) freq = 10000;
  }
}

void tunegen() {
  si5351.set_freq((freq + (interfreq * 1000ULL)) * 100ULL, SI5351_CLK0);
  //si5351.set_freq_manual((freq + (interfreq * 1000ULL)) * 100ULL, pll_freq, SI5351_CLK0);
}

void displayfreq() {
  unsigned int m = freq / 1000000;
  unsigned int k = (freq % 1000000) / 1000;
  unsigned int h = (freq % 1000) / 1;

  char buffer[15] = "";
  if (m < 1) {
    lcd.setCursor(5, 15); sprintf(buffer, "%003d.%003d", k, h);
  }
  else if (m < 100) {
    lcd.setCursor(5, 15); sprintf(buffer, "%2d.%003d.%003d", m, k, h);
  }
  else if (m >= 100) {
    unsigned int h = (freq % 1000) / 10;
    lcd.setCursor(5, 15); sprintf(buffer, "%2d.%003d.%02d", m, k, h);
  }
  lcd.print(buffer);
}

void setstep() {
  switch (stp) {
    case 1:
      stp = 2;
      fstep = 10;
      break;
    case 2:
      stp = 3;
      fstep = 100;
      break;
    case 3:
      stp = 4;
      fstep = 1000;
      break;
    case 4:
      stp = 5;
      fstep = 10000;
      break;
    case 5:
      stp = 6;
      fstep = 100000;
      break;
    case 6:
      stp = 1;
      fstep = 1000000;
      break;
  }
}

void layout() {
  lcd.setCursor(8, 1);
  lcd.print("TS:");
  if (stp == 2) lcd.print("10Hz   "); 
  if (stp == 3) lcd.print("100Hz  "); 
  if (stp == 4) lcd.print("1k    ");
  if (stp == 5) lcd.print("10k    "); 
  if (stp == 6) lcd.print("100k   "); 
  if (stp == 1) lcd.print("1M    ");
  
  lcd.setCursor(0, 1);
  lcd.print("IF:");
  lcd.print(interfreq);
  lcd.print("k");
  lcd.setCursor(0, 2);
  lcd.print("S-METR:");
  lcd.setCursor(0, 3);
  lcd.print("FREQ:");
  /*
  lcd.setCursor(15, 0);
  if (interfreq == 0) lcd.print("TX");
  if (interfreq != 0) lcd.print("RX");
  */
}

      void initBar1() {
        // необходимые символы для работы
        byte right_empty[8] = {0b11111,  0b00001,  0b00001,  0b00001,  0b00001,  0b00001,  0b00001,  0b11111};
        byte left_empty[8] = {0b11111,  0b10000,  0b10000,  0b10000,  0b10000,  0b10000,  0b10000,  0b11111};
        byte center_empty[8] = {0b11111, 0b00000,  0b00000,  0b00000,  0b00000,  0b00000,  0b00000,  0b11111};
        lcd.createChar(0, left_empty);
        lcd.createChar(1, center_empty);
        lcd.createChar(2, right_empty);
       }

    void fillBar1(byte start_pos, byte row, byte bar_length, byte fill_percent) {
      byte infill = round((float)bar_length * fill_percent / 100);
      lcd.setCursor(start_pos, row);
      if (infill == 0) lcd.write(0);
      else lcd.write(255);
      for (int n = 1; n < bar_length - 1; n++) {
        if (n < infill) lcd.write(255);
        if (n >= infill) lcd.write(1);
      }
      if (infill == bar_length) lcd.write(255);
      else lcd.write(2);
     }

    void ProcessNumber(char key)
    {
      if(counter >= maxLength)
      return; 
      counter++;
      input += key;
      lcd.setCursor(0,0);
      lcd.print(input);
       
    }

    bool IsNumber(char key)
      {
        if(key >= '0' && key <= '9')
         return true;
         return false;
      }

    void Reset()
      {
        counter = 0;
        input.remove(0);
        lcd.setCursor(0, 0);
        lcd.print("                    ");
      }

void setup() {
  Wire.begin();
    
  lcd.backlight();          // включаем подсветку
  lcd.begin(20, 4);         // Initialize and clear the lcd
  lcd.clear();  

   pinMode(ENCODER_A, INPUT );
   pinMode(ENCODER_B, INPUT );
   pinMode(tunestep, INPUT );

  si5351.init(SI5351_CRYSTAL_LOAD_8PF, 0, cal);
  si5351.output_enable(SI5351_CLK0, 1);                  //1 - Enable / 0 - Disable CLK
  si5351.output_enable(SI5351_CLK1, 0);
  si5351.output_enable(SI5351_CLK2, 0);
  si5351.drive_strength(SI5351_CLK0, SI5351_DRIVE_2MA);  //Output current 2MA, 4MA, 6MA or 8MA

  PCICR |= (1 << PCIE2);
  PCMSK2 |= (1 << PCINT18) | (1 << PCINT19);
  sei();

  initBar1(); // инициализация символов для отрисовки s-метра.
  stp = 5; //  дефолтная установка шага перестройки
  setstep();
  layout();
  displayfreq();
}

void loop() {

char key = keypad.getKey();

 if (key) {
    if(IsNumber(key))
    {
      ProcessNumber(key);
    }
    else if(key == '#' && input.length() > 0)
    {
      if (input.length() <= 4){
      int o_input = 7 - input.length();
            for (int i = 0; i < o_input; i++) {
                input += 0;
            }
          } else {
            int o_input = 8 - input.length();
        for (int i = 0; i < o_input; i++) {
            input += 0;
          }
      }
      freq = input.toInt();
      Reset();
    }
    else if(key == '*')
    {
      Reset();
    }
  }

     if (millis() - time_now > 200){ // время обновления S-метра в mc
      time_now = millis(); 
      int perc = map(analogRead(sMetr), 0, 1023, 0, 100);
      fillBar1(7, 2, 10, perc);
      }
       
    if (freqold != freq) { 
      time_now = millis();
      tunegen();
      freqold = freq;
    }
  
    if (digitalRead(tunestep) == LOW) {
      time_now = (millis() + 300);
      setstep();
      delay(300);
    }
  
    if ((time_now + period) > millis()) {
      displayfreq();
      layout();
    }
}
