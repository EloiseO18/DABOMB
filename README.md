#include "mbed.h"
#include "stdio.h"
#include "math.h"
#include "ADXL362.h"

/*Create objects for input/output */
Serial pc(USBTX, USBRX); /*serial interface*/
DigitalOut led1(LED1);/* LED1 */
DigitalOut led2(LED2);/* LED2 */
DigitalOut led3(LED3);/* LED3 */
DigitalOut led4(LED4);/* LED4 */
ADXL362 adxl362(p11, p12, p13, p10);  // Accelerometer (mosi, miso, sclk, cs)


//Define Global Variables
int N;
float T;
float f;

// Main Loop
int main()
{
    /*Define initial variable values */
    N = 50;
    T = 0.1;
    f = 1/T;
       
    //Access external paramater value file************************************************************************************************              
    //parameter values stored in "set_parameters"
	FILE *setp;
	
	setp = fopen(set_parameters.txt)
    
    /*initialise data capture arrays ready to be filled */            
    float x_stored[N];
    float y_stored[N];
    float z_stored[N]; 
    float time[N];
    
    /*define some useful variables*/
    int x_alignment,y_alignment,z_alignment;
    float x_max,y_max,z_max,x_min,y_min,z_min;
    
    /* Informative text of the process */          
    /*"Please note:
    "When the x-axis alignes to the direction of gravity LED1 will switch on,
    When the y-axis alignes to the direction of gravity LED2 will switch on,
    When the z-axis alignes to the direction of gravity LED3 will switch on,
    Within an error of +/- 10%        
    The data capture will last for aproximatly %f seconds. \n", N*T,
    The lights will flash when complete.*/
    
    /* Define initial values of alignment/max/min variables */
    x_alignment=0;
    y_alignment=0;
    z_alignment=0;
    x_max=0;
    y_max=0;
    z_max=0;
    x_min=0;
    y_min=0;
    z_min=0;
        
    // local variables
    int8_t x8 = 0;
    int8_t y8 = 0;
    int8_t z8 = 0;  
    uint8_t reg;
  
    // set up SPI interface
    adxl362.init_spi();
    // Set up accelerometer
    adxl362.init_adxl362();
    
    // Check settings
    reg = adxl362.ACC_ReadReg(FILTER_CTL);
    pc.printf("FILTER_CTL = 0x%X\r\n", reg);
    pc.printf("\nComencing Data capture...\n");

    //recive data
    for(int i=0;i<N;i++)
    { 
        adxl362.ACC_GetXYZ8(&x8, &y8, &z8); // Fetch sample from ADXL362
        /* USE "ACC_GetXYZ8" ??? */
        // Converting the raw 8bit value to a value in terms of m/s^2
        // multiplying factor or '2*9.8/128' sinse +128 equates to +2g and -128 to -2g
        x_stored[i] = (((float)x8*49.0)/320.0); 
        y_stored[i] = (((float)y8*49.0)/320.0);
        z_stored[i] = (((float)z8*49.0)/320.0);
        time[i] = i*T;
        
        /* Series of If statments to update the maximum and minimum values of x,y,z   */
        if(x_stored[i] > x_max)
            {
                x_max = x_stored[i];
            }    
        if(y_stored[i] > y_max)
            {
                y_max = y_stored[i];
            }
        if(z_stored[i] > z_max)
            {
                z_max = z_stored[i];
            }
       
        if(x_stored[i] < x_min)
            {
                x_min = x_stored[i];   
            }
        if(y_stored[i] < y_min)
            {
                y_min = y_stored[i];   
            }
        if(z_stored[i] < z_min)
            {
             z_min=z_stored[i];   
            }

        //If statments to controll the axis alignment condidions and effects
        //This is based on the idea that if 2 of the axis have a value of ~0 then the final axis must me in alignment with the direction of gravity
        
        /* Turn on LED1 when within +/-10% of X-axis */
        if((y8>=-12.8 && y8<=12.8) && (z8>=-12.8 && z8<=12.8)) 
        {
            led1=1;
            x_alignment=x_alignment+1;
        }
        else
        {
            led1=0;
        }

        /* Turn on LED1 when within +/-10% of Y-axis */
        if((x8>=-12.8 && x8<=12.8) && (z8>=-12.8 && z8<=12.8))
        {
            led2=1;
            y_alignment=y_alignment+1;
        }
        else
        {
            led2=0;
        }

        /* Turn on LED1 when within +/-10% of Z-axis */
        if((y8>=-12.8 && y8<=12.8) && (x8>=-12.8 && x8<=12.8))
        {
            led3=1;
            z_alignment=z_alignment+1;
        }
        else
        {
            led3=0;
        }

		
            wait(T);
    } 
    
    /*Data capture complete. */
    /* flash light to signal the end */
    led1=0;
    led2=0;
    led3=0; 
    led4=0;
    wait(0.1);                        
    led1=1;
    led2=1;
    led3=1; 
    led4=1;                                                       
    wait(0.1);                        
    led1=0;
    led2=0;
    led3=0; 
    led4=0;
    
    /*DATA VALUES FOR MATLAB ACCESS */
    //The number of times the X-axis aligned was:   'x_alignment'
    //The number of times the Y-axis aligned was:   'y_alignment'
    //The number of times the Z-axis aligned was:   'z_alignment'
    //The maximim values were for each axis:         x_max, y_max, z_max
    //The minimum values were for each axis:         x_min, y_min, z_min
    //Sampling Frequency value:                      f
     
}




//Set parameters Via MATLAB ready for the C-script to run
void set_parameters()
{
	
	FILE *read_file;
	
	
	
	
	
	
	
    
    
    
