# Portable snake implementation
A driver that processes a snake game and gives the user the ability to flush the current frame buffer with a self defined function. Mainly aimed at the usage on embedded platforms. See the main.c file for a example usage. 

## WIP
  - Still uses parts of Standard library.
  - ~~Not all possible errors are handled properly.~~
  - ~~Square drawing algorithm can be improved speed-wise.~~
  - No sound support (yet).
  - ~~No custom color support (yet).~~
## Example usage
A FPGA system connected to a VGA screen using a pixel buffer at location FRAME_BUFFER_BASE. Full repository: [FPGA Nios snake](https://github.com/MartBent/fpga-nios-snake)

### Code
```C
#include "snake.h"

volatile int edge_capture;
static direction_t direction = down;

void delay(unsigned long msec) {
    usleep(msec*300);
}

void display_flush(const u8* grid, const unsigned long resolution) {
    //Not needed since the frame buffer pointer is passed
}

u8 rnd() {
    return (rand() % 30) + 1;
}

void display_score(u8 score) {
    IOWR_ALTERA_AVALON_PIO_DATA(SCORE_PIO_BASE, score);
}

direction_t read_direction() {
    return direction;
}

static void interrupt(void* c) {
    volatile int* edge_capture_ptr = (volatile int*)c;
    *edge_capture_ptr = IORD_ALTERA_AVALON_PIO_EDGE_CAP(BTN_PIO_BASE);
    IOWR_ALTERA_AVALON_PIO_EDGE_CAP(BTN_PIO_BASE, 0);

    //Determine direction according to which key is pressed
    switch(edge_capture) {
        case 8:
	    direction = up;
	    break;
	case 4:
	    direction = right;
	    break;
	case 2:
	    direction = left;
	    break;
	case 1:
	    direction = down;
	    break;
	}
	
    //No debounce required in this application
}

int main(void) {

	//Register interrupts
	IOWR_ALTERA_AVALON_PIO_IRQ_MASK(BTN_PIO_BASE, 0xf);
	IOWR_ALTERA_AVALON_PIO_EDGE_CAP(BTN_PIO_BASE, 0x0);
	void* edge_capture_ptr = (void*) &edge_capture;
	alt_ic_isr_register(BTN_PIO_IRQ_INTERRUPT_CONTROLLER_ID, BTN_PIO_IRQ, interrupt, edge_capture_ptr, NULL);

	//Seed RNG
	srand(1235);

	//Play snake
	while(1) {
	    snake_driver_t driver;
	    
	    driver.delay_function_cb = delay;
	    driver.display_frame_cb = display_flush;
	    driver.display_score_cb = display_score;
	    driver.random_number_cb = rnd;
	    driver.read_direction_cb = read_direction;
	    driver.resolution = 512;
	    driver.snake_length = 32;
	    driver.snake_color = 0xFF; //White
	    driver.background_color = 0x00; //Black
	    driver.food_color = 0x5A; //Gray
	    driver.frame_buffer = (u8*)FRAME_BUFFER_BASE;
	    
	    printf("Starting snake...\n");
	    char* msg = snake_play(&driver);
	    printf(msg);
	}
}
```
### Output

Playing snake on a VGA monitor

![Snake](https://i.imgur.com/IClmkOl.gif)
