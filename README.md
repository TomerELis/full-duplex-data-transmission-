# full-duplex-data-transmission-

This project implements a USART (Universal Synchronous and Asynchronous Receiver-Transmitter) communication protocol using Arduino. It simulates data transmission (TX) and reception (RX) between devices with parity checking and error handling.


# How It Works
The USART communication in this project is implemented through two main processes: transmitting (TX) and receiving (RX).

The transmitter (TX) starts by preparing the data to send. It begins with a start bit, followed by the data bits, a parity bit (calculated using __builtin_popcount() for odd parity), and ends with a stop bit. Each bit is sent sequentially, with precise timing controlled by millis(). The transmitter ensures that all bits are sent properly before transitioning back to idle.

The receiver (RX) continuously monitors the RX pin for a start bit. Once detected, it reads the data bits, validates each bit using majority voting logic, and verifies the parity bit to ensure data integrity. If the parity or any other bit does not match the expected pattern, the receiver transitions to an error state, resets necessary parameters, and waits for the next frame.

Both TX and RX processes handle timing using BIT_TIME to ensure synchronization between the transmitter and receiver. This guarantees that the data transmission and reception are both accurate and robust, even in the presence of timing variations.

# link to a scheme
https://ibb.co/c8TCXDS

# Full Code

//Work of Tomer Elis

#define BAUD_RATE 9600
#define TX_PIN 12
#define RX_PIN 13
#define Min 400
#define Max 500
#define BIT_TIME 20
//


//variables TX
#define buffer 'B'

#define IDLE 1
#define START 2
#define Data 3
#define Party_bit 4
#define STOP 5
#define COM_L2 6
#define ERROR 7
#define Length_Data 8

static int count=0, state = IDLE,tx_busy=0, pairty=0;
static long int rands = 0;
static float ref_random=millis(),ref_BT = 0; //status time
static unsigned long Time; //del (?)
static bool first_flag = true;

//variables RX

#define ignore 3
#define error 4
#define SAMP_NUM 3.0
#define checker_delta 0b1110
#define zero_delta 0
#define one_delta 14

static int rx_busy=0, y=0, buffer_delta = 14,counter=0,delta_pin=0; //****
static int counter_delta=0,B=0;
static float ref_error=0, delta = BIT_TIME/(SAMP_NUM+2);
static int   state_rx = IDLE, message=0;
static float ref_delta=millis(); //status time



//'buffer' is the input(sending by TX)
//and 'message' is the output (recieve in RX)

void setup()
{
  pinMode(TX_PIN, OUTPUT);
  pinMode(RX_PIN, INPUT);
  digitalWrite(TX_PIN,1);
 
  Serial.begin(BAUD_RATE);
}

void loop()
{

    Usart_TX();
  Usart_RX();
 
}



void Usart_TX()
{
switch (state) {
  case IDLE:
  tx_busy=1; //for Lab 3
 
  first_flag = true;
  count=0;
  pairty = (1 + __builtin_popcount(buffer))%2;
  //Serial.print(pairty);
  //Serial.print(" ");
  state = START;
  //calc pairty bit using __builtin_popcount = sum the bits of data
  //we added 1 in order to make odd pairty
    break;
 
  case START:
  if (first_flag)
  {
    first_flag = false;
  digitalWrite(TX_PIN, 0); //Sending start Bit
    ref_BT = millis();
  }
  else if (Check_BT())
  {
  state = Data;
    first_flag = true;
  }
    break;
 
  case Data:
  if (count == Length_Data) //finish to sent the data
    state = Party_bit;
   
  else if (first_flag){
  first_flag = false;
digitalWrite(TX_PIN,bitRead(buffer, count));
     //the above line write the bits of the Data in index count
    ref_BT = millis();                  
  }
  else if (Check_BT())
  {
count++;  //continue to send until 1 Bit Time as passed
  first_flag = true;
  }
 
    break;
 
  case Party_bit:
 
  if (first_flag){
    digitalWrite(TX_PIN, pairty); //we are sending the pairty bit
  first_flag = false;
    ref_BT = millis();      
  }
    else if (Check_BT())
  {
    first_flag = true;
  state = STOP;
    ref_BT = millis();
  }
    break;
 
  case STOP:

  if (first_flag){
  digitalWrite(TX_PIN, 1); // Sending the stop bit
    first_flag = false;
  }
 
  else if (Check_BT())
  {
  state = COM_L2;
    ref_random = millis();
  rands  = random(Min,Max);
  }
    break;
 
  case COM_L2:
  if (millis() - ref_random >= rands)
  {
    state = IDLE;
  }
 
    break;
 
  default:
    Serial.print("HELLO THIS IS SICSIC I AM THE DEFAULT KING!!!");
    break;
}
}



void Usart_RX()
{
  //y = Read_bit();

switch (state_rx) {
  case IDLE:

  if (digitalRead(RX_PIN)== 0)
  {
    state_rx = START;
    rx_busy = 1;
    //****************reset all parameters of RX + WAIT BIT  T
    counter = 0;
  }
    break;
 
  case START:
  y = Read_bit();
  if (y == ignore)
    break;
  else if (y == 0) //if the bit is zero the start bit is correct
    state_rx = Data;
  else //if (y == 1 or y == error) //the bit is corrupted --> error
    state_rx = error;
   //when we get "ignore" we'll just ignore it (else)
    break;
 
  case Data:
  //Serial.print("data");
  if (counter == Length_Data) //all the data been readed
  {
    //Serial.print("#");
    //Serial.print(message);
    state_rx = Party_bit;
    counter++; //counter point on the party_bit
  }
  else
  {
    y = Read_bit();
    if (y == ignore)
    break;
    else if (y==0 or y==1) //the Data bit at index counter is 0/1
    {
     
      bitWrite(message,counter,y);//y is the data bit from the chanale
      counter++;
    }
    else //if(y==error)  
    {
     
     state_rx = ERROR;
      counter++; // counter point of the bit that made the error

  }
   //when we get "ignore" we'll just ignore it (else)
    break;
 
  case Party_bit:
    y = Read_bit();
   
    if (y == ignore)
    break;
   
    else if (y == 0 or y == 1)
    {
      //Serial.print(y);
      if (((__builtin_popcount(message) + y)%2) == 0) // checkif the data is even
        state_rx = ERROR;
      else // the data is odd - which is good
      {      
        state_rx = STOP;
        counter++;
      }
    }
    else //if (y == error)
      state_rx = ERROR;
   //when we get "ignore" we'll just ignore it (else)
    break;
 
  case STOP:
   
    y = Read_bit();
      if (y == ignore)
    break;
   
    else if (y == 1)
    {        
      //Serial.print(" STOP :( \n");
      state_rx = COM_L2;
      counter++;
    }  
   
     else //if (y == error or y==0)        
      state_rx = ERROR;
    //when we get "ignore" we'll just ignore it (else)
    break;
 
  case COM_L2:
   
    state_rx = IDLE;
    Serial.print(char(message));
    Serial.print("\n");
    break;
   
 case ERROR:
   
    Serial.print("ERROR");
    if (millis() - ref_error >= BIT_TIME*(2+Length_Data-counter))
//we have to wait the BIT TIME * the amount of 'All the frame bits'
//minus the number of BITS we been thorugh minus
//the current bit.
    //frame Length = 3 + Length_Data (star + paity bit+ stop + Length_Data )
     // BIT TIME*(Frame Length - 1 -counter)
      state_rx = IDLE;
    break;

   
 
  default:
    Serial.print("HELLO THIS IS SICSIC I AM THE DEFAULT KING!!!");
    break;
}
}
}


//------------help functions---------------------

bool Check_BT(){ // check that 1 bit time as passed
  if (millis()-ref_BT>= BIT_TIME)
  return true;
  return false;
}

int Read_bit() // read bit 5 times and check the true value of bit
{
  if (millis()-ref_delta < delta)
    return ignore;
  else
  {
    //Serial.print("$ ");
    bitWrite(buffer_delta, counter_delta, digitalRead(RX_PIN));
    ref_delta = millis();
    counter_delta++;
    //Serial.print(digitalRead(RX_PIN));
    if (counter_delta < 5)
      return ignore;
    else
    {
      //Serial.print(" ");
      counter_delta = 0;
      B = buffer_delta & checker_delta;
      if (B == zero_delta)
        return 0;
      else if (B == one_delta)
        return 1;
      return error;
     
    }
  }
 
}
