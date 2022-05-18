#define range_pin 2      // output of comparator
#define input_voltage A0 // output of relay

const int N = 1024;      // N of Arduino's analog pin 
const double res1 = 10/(double)N;  // +-5 resolution 
const double res2 = 1/(double)N;   // +-0.5 resolution 
const int zero_volt = N/2;		   // =512, middle value, 0[v]

double voltage;	// variable for voltage value
double ring_buffer[5] = {0,0,0,0,0}; //ring buffer for 5 voltages
int idx = 0;
int cnt = 0; 	//cnt for 5 secs
double sqr_std; //standard deviation
double mean = 0;
int ar;   // analog read value
bool range_5;  // true if current range is +-5[v]

void calc_std(); // clac the squared value of standard deviation


void setup()
{
  	pinMode(range_pin, INPUT); //set switch pin as input
	Serial.begin(9600);
  	Serial.println("Startup system...");
  	// TIMER 1 for interrupt frequency 1 Hz:
    cli(); // stop interrupts
    TCCR1A = 0; // set entire TCCR1A register to 0
    TCCR1B = 0; // same for TCCR1B
    TCNT1  = 0; // initialize counter value to 0
    // set compare match register for 1 Hz increments
    OCR1A = 62499; // = 16000000 / (256 * 1) - 1 (must be <65536)
    // turn on CTC mode
    TCCR1B |= (1 << WGM12);
    // Set CS12, CS11 and CS10 bits for 256 prescaler
    TCCR1B |= (1 << CS12) | (0 << CS11) | (0 << CS10);
    // enable timer compare interrupt
    TIMSK1 |= (1 << OCIE1A);
    sei(); // allow interrupts
}

ISR(TIMER1_COMPA_vect){
   //interrupt commands for TIMER 1 here
  	cnt++;
  	if(cnt == 5) // after 5 secs
    {
      for(idx = 0; idx < 5; idx ++)
      {
        ar = analogRead(input_voltage); //read analog voltage
        range_5 = digitalRead(range_pin);
        if(range_5) //range is +-5[v]
          voltage = res1 * (double)(ar-zero_volt);
        else     //range is +-0.5[v]
          voltage = res2 * (double)(ar - zero_volt);   
        ring_buffer[idx] = voltage; //add value to ring buffer
        mean += voltage;
      }

      
      mean = mean/5; //calc mean of ring buffer values
      calc_std(); // squared value of standard deviation
      Serial.print("V = ");
      if(abs(mean) > 0.5) // for +-5[v]
      {
        Serial.print(mean);
        Serial.println(" [V]");
      }
      else
      {
        Serial.print(mean*1000);  // 500.00[mV]
        Serial.println(" [mV]");
      }
      Serial.print("squared standard deviation: ");
      Serial.print(sqr_std);
      Serial.println(" [V]");
      mean = 0; // reset mean and idx for next sampels
      idx = 0;
      cnt = 0;
    }
}


void loop() {}
  
void calc_std()
{
  double numerator = 0; 
  //numerator = sum((xi - mean(x))^2)
  for(int i = 0; i<5; i++)
  	numerator += (ring_buffer[i] - mean)*(ring_buffer[i] - mean); 
  sqr_std = numerator/5;
  //std^2 = sum((xi - mean(x))^2)/N

}
