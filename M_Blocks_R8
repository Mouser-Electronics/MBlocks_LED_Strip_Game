/*
	LED STRIP Block GAME
    	Copyright (C) <2016>  <Ryan L. Prewitt> <Mouser Electronics>

   	This program is free software: you can redistribute it and/or modify
  	it under the terms of the GNU General Public License as published by
   	the Free Software Foundation, either version 3 of the License, or
    	any later version.

    	This program is distributed in the hope that it will be useful,
    	but WITHOUT ANY WARRANTY; without even the implied warranty of
    	MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    	GNU General Public License for more details.

    	You should have received a copy of the GNU General Public License
    	along with this program.  If not, see <http://www.gnu.org/licenses/>.
	

	http://www.mouser.com/applications/led-blocks-game/

      	ORIGINALLY WRITTEN BY:  YAE YEONG BAE (FOR ARDUINO AND SINGLE COLOR DOT MATRIX DISPLAY)
      	MODIFIED FOR USE WITH DIGILENT MAX32 DEVELOPMENT PLATFORM AND WS2812 LED STRIP

      
*/



#include <PICxel.h>

#define LED_pin 5
#define LED_total_count 900
#define number_of_Block_LEDs 300
#define Block_row_size 10
#define Block_column_size 30
#define button (PORTDbits.RD8 << 0) + (PORTDbits.RD9 << 1) + (PORTDbits.RD10 << 2) + (PORTDbits.RD11 << 3)

////////////////////////////////////////////////////////////////////////////////////////////////////////
#define CHANGE_HEAP_SIZE(size) __asm__ volatile ("\t.globl _min_heap_size\n\t.equ _min_heap_size, " #size "\n")
CHANGE_HEAP_SIZE(0x2000);
extern __attribute__((section("linker_defined"))) char _heap;
extern __attribute__((section("linker_defined"))) char _min_heap_size;
///////////////////////////////////////////////////////////////////////////////////////////////////////

//*************************************************************************************************//
//  PGM VARIABLES AND INITS
//*************************************************************************************************//

unsigned char blocktype;
unsigned char blockrotation;
unsigned char blockcolor;
unsigned int x_axis = 0;
unsigned int y_axis = 0; 
unsigned int LED_index[Block_column_size];
unsigned int number_of_Block_strips = number_of_Block_LEDs / Block_column_size;
unsigned int current_Block_strip = 0;
const unsigned int Jup_pin = 12 ;
const unsigned int Jdown_pin = 13;
const unsigned int Jleft_pin = 14;
const unsigned int Jright_pin = 15;
unsigned int previous_button;
bool early_break = 0;
unsigned int delaytime = 0;
unsigned int cleared_index[Block_row_size] = {0};
PICxel strip(LED_total_count, LED_pin, HSV);
unsigned int run_status = 0;
bool gameover_flag = 0;
int row;
unsigned int total_lines_cleared = 0;
unsigned int difficulty = 150;
short drop_difficulty_count = 0;
short dropcount = 2;

//calculated using http://www.rapidtables.com/convert/color/rgb-to-hsv.htm
//Hue = Degree Angle * 4.2635 ; Saturation = % of 255 ; Value(brightness) % of 255
uint16_t disp_colors [10][3] = {
  {0,0,0},  //Color 0 - OFF
  {0,255,100},  //Color 1 - RED
  {110,255,100},  //Color 2 - ORANGE
  {255,255,100},  //Color 3 - YELLOW
  {512,255,100},  //Color 4 - GREEN
  {896,255,100},  //Color 5 - BLUE
  {1500,255,120},  //Color 6 - PINK
  {768,255,100},  // Color 7 - LT BLUE
  {0,0,80},  //Color 8 - WHITE
  {1084,255,40},  //Color 9 - DARK BLUE 
};

uint16_t logo_colors[2][3] = {
  {1024,255,15},
  {0,0,80},
};

bool full_array[Block_column_size][Block_row_size] = {0};
bool used_full_array[Block_column_size][Block_row_size] = {0};
unsigned int color_array[Block_column_size][Block_row_size] = {0};
bool mini_upper[Block_column_size / 2][Block_row_size] = {0};
bool mini_lower[Block_column_size / 2][Block_row_size] = {0};
bool block[Block_row_size][Block_column_size+2] = {0};
bool pile[Block_row_size][Block_column_size] = {0};
bool disp[Block_row_size][Block_column_size] = {0};
unsigned int temp_Block_datamap[300];
unsigned int Block_datamap[300];
unsigned int next_piece = 0;

const unsigned int master_data_map[LED_total_count] ={
9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,
9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,
9,1,1,1,1,1,0,0,0,0,0,0,0,0,0,9,1,1,1,1,1,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,1,9,0,0,0,0,0,0,0,0,0,1,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,0,0,9,1,0,0,0,0,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,1,1,1,1,0,9,0,0,0,0,0,0,0,0,0,1,1,1,1,1,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,0,0,9,0,0,0,0,0,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,1,1,1,1,1,9,0,0,0,0,0,0,0,0,0,1,1,1,1,1,9,
9,1,0,0,0,0,0,0,0,0,0,0,0,0,0,9,1,0,1,0,1,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,1,0,1,0,1,9,0,0,0,0,0,0,0,0,0,1,1,1,1,0,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,0,0,9,0,0,0,0,0,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,1,1,0,1,1,9,0,0,0,0,0,0,0,0,0,1,1,1,1,1,9,
9,1,0,1,0,1,0,0,0,0,0,0,0,0,0,9,0,0,1,0,0,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,1,1,0,1,1,9,0,0,0,0,0,0,0,0,0,1,0,1,0,1,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,0,0,9,0,0,0,0,0,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,1,9,0,0,0,0,0,0,0,0,0,1,0,0,1,0,9,
9,1,0,1,0,1,0,0,0,0,0,0,0,0,0,9,1,1,1,1,1,0,0,0,0,0,0,0,0,9,
9,0,0,0,0,0,0,0,0,0,0,0,0,1,9,0,0,0,0,0,0,0,0,0,0,1,0,0,1,9,
9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9,9
};

const bool Logo_Datamap[LED_total_count] ={
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,
0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,
0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,
0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0  
};

const unsigned int default_data_map[LED_total_count] ={0};
volatile unsigned int output_data_map[LED_total_count];

// Number Characters:
// 0
bool num_char0[5][8]= {
{0,1,1,1,1,1,1,0},
{1,0,0,0,1,0,0,1},
{1,0,0,0,1,0,0,1},
{1,0,1,0,0,0,0,1},
{0,1,1,1,1,1,1,0}
  };

// 1
bool num_char1[5][8]= {
{1,0,0,0,0,0,0,0},
{1,0,0,0,0,0,0,1},
{1,1,1,1,1,1,1,1},
{0,0,0,0,0,0,0,1},
{1,0,0,0,0,0,0,0}
};

// 2
bool num_char2[5][8]= {
{1,1,0,0,0,1,1,0},
{1,0,0,0,0,1,0,1},
{1,0,0,1,0,0,0,1},
{1,0,0,1,0,0,0,1},
{1,0,0,0,0,1,1,0}
};

// 3
bool num_char3[5][8]= {
{0,1,0,0,0,0,1,0},
{1,0,0,0,1,0,0,1},
{1,0,0,1,0,0,0,1},
{1,0,0,0,1,0,0,1},
{0,1,1,1,1,1,1,0}
};

// 4
short num_char4[5][8]= {
{0,0,0,0,1,1,0,0},
{0,1,0,1,0,0,0,0},
{0,0,0,0,1,0,0,1},
{1,1,1,1,1,1,1,1},
{0,0,0,0,1,0,0,0}	
};

// 5
bool num_char5[5][8]= {
{0,1,0,0,1,1,1,1},
{1,0,0,1,0,0,0,1},
{1,0,0,0,1,0,0,1},
{1,0,0,1,0,0,0,1},
{0,1,1,1,0,0,0,1}
};

// 6
bool num_char6[5][8]= {
{0,1,1,1,1,1,1,0},
{1,0,0,1,0,0,0,1},
{1,0,0,0,1,0,0,1},
{1,0,0,1,0,0,0,1},
{0,1,1,1,0,0,1,0}	
};

// 7
short num_char7[5][8]= {
{0,0,0,0,0,0,0,1},
{1,0,0,0,0,0,0,0},
{1,1,0,0,0,0,0,1},
{1,0,0,0,1,1,0,0},
{0,0,0,0,1,1,1,1}
};

// 8
bool num_char8[5][8]= {
{0,1,1,1,1,1,1,0},
{1,0,0,1,0,0,0,1},
{1,0,0,0,1,0,0,1},
{1,0,0,1,0,0,0,1},
{0,1,1,1,1,1,1,0}	
};

// 9
short num_char9[5][8]= {
{0,0,0,0,1,1,1,0},
{1,0,0,0,1,0,0,1},
{1,0,0,1,0,0,0,1},
{1,0,0,0,1,0,0,1},
{0,1,1,1,1,1,1,0}	
};

//*************************************************************************************************//
//  MASTER SETUP ROUTINE
//*************************************************************************************************//

void setup(){
  pinMode(Jup_pin, INPUT); //rotate
  pinMode(Jdown_pin, INPUT); //down
  pinMode(Jleft_pin, INPUT); //left
  pinMode(Jright_pin, INPUT); //right
  pinMode(5, OUTPUT); //data
  Serial.begin(9600);
  strip.begin();
  //send_strip_data(0);
  delaytime = 250;  //default
  //flash_m_logo_on();
}

//*************************************************************************************************//
//  MAIN PGM LOOP
//*************************************************************************************************//

void loop(){
  while (run_status == 0)
  {
  send_strip_data(2);// send logo
  check_start();
  }
    switch (button)
    {
    case 0:	//  Move Down
      previous_button = button;
      delaytime = difficulty;
      if (early_break == 0)
      {
        movedown();
      }
      else
      {
        early_break = 0;
      }
      Blockmatrix_update();
      break;

    case 32:		//rotate
      previous_button = button;
      rotate();
      delaytime = difficulty+50;
      Blockmatrix_update();
	  dec_dropcount();
      break;

    case 16:		//movedown
      previous_button = button;
              switch (difficulty)
        {
          case 150:
          delaytime = difficulty/2;
          break;
          case 100:
          delaytime = difficulty/2;
          break;
          case 75:
          delaytime = difficulty/2;
          break;
          case 50:
          delaytime = 30;
          break;
          case 25:
          delaytime = difficulty;
          break;
        }
      movedown();
      Blockmatrix_update();
      break;

    case 2:	//moveleft
      previous_button = button;
      moveleft();
      delaytime = difficulty;
      Blockmatrix_update();
	  dec_dropcount();
      break;

    case 1:	//moveright
      previous_button = button;
      moveright();
      delaytime = difficulty;
      Blockmatrix_update();
	  dec_dropcount();
      break;
    }
    
    for (int dly = 0; dly < delaytime; dly++)
    {
      if (previous_button != button)
      {
        early_break = 1;
        break;
      }
      delay(1);
    }
}// end of loop

//*************************************************************************************************//
//  FUNCTIONS
//*************************************************************************************************//
void check_start(){
    if (( PORTD > 65527 ) && ( run_status == 0))
  {
    Serial.println("New Game Started!");
    set_default_master_data_map();
    //run_status = 1;
    gameover_flag = 0;
    drop_new_block();
    updateLED;
    generate_next_piece_gfx();
    run_status = 1;
    delay(500);
  }
}

//*************************************************************************************************//
void dec_dropcount(){
if (button > 0)
  {
  dropcount--;
  //Serial.println(dropcount);
  }
  if (dropcount == 0)
  {
  movedown();
  dropcount = 2;
  return;
  }
else if (button == 0)
  {
  dropcount = 2;
  }
}

//*************************************************************************************************//
bool moveleft(){
  if (space_left())
  {
    int i;
    int j;
    for (i=0;i<Block_row_size-1;i++)
    {
      for (j=0;j<Block_column_size;j++)      
      {
        block[i][j]=block[i+1][j];
      }
    }
    for (j=0;j<Block_column_size;j++)      
    {
      block[Block_row_size-1][j]=0;
    }    
    updateLED();
    return 1;
  }
  return 0;
}

//*************************************************************************************************//
bool moveright(){
  if (space_right())
  {
    int i;
    int j;
    for (i=Block_row_size-1;i>0;i--)
    {
      for (j=0;j<Block_column_size;j++)      
      {
        block[i][j]=block[i-1][j];
      }
    }
    for (j=0;j<Block_column_size;j++)      
    {
      block[0][j]=0;
    }    
    updateLED(); 
    return 1;   
  }
  return 0;
}

//*************************************************************************************************//
void movedown(){
  if (space_below())
  {
    //move down
    int i;
    for (i=Block_column_size-1;i>=0;i--)
    {
      int j;
      for (j=0;j<Block_row_size;j++)
      {
        block[j][i] = block[j][i-1];
      }
    }
    for (i=0;i<Block_row_size-1;i++)
    {
      block[i][0] = 0;
    }
  }
  else
  {
    //merge and new block
    int i;
    int j;    
    for (i=0;i<Block_row_size;i++)
    {
      for(j=0;j<Block_column_size;j++)
      {
        if (block[i][j])
        {
          pile[i][j]=1;
          block[i][j]=0;
        }
      }
    }
    check_gameover(); 
    clear_lines();
    if (gameover_flag != 1)
    {
      drop_new_block();
      generate_next_piece_gfx();
    }
  }
  if (gameover_flag != 1);
  {
    updateLED(); 
  }
}

//*************************************************************************************************//
void rotate(){
  //skip for square block(3)
  if (blocktype == 3) return;
  int xi;
  int yi;
  int i;
  int j;
  //detect left
  for (i=Block_row_size-1;i>=0;i--)
  {
    for (j=0;j<Block_column_size;j++)
    {
      if (block[i][j])
      {
        xi = i;
      }
    }
  }
  //detect up
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        yi = i;
      }
    }
  }  
  if (blocktype == 0)
  {
    if (blockrotation == 0) 
    {
      if (!space_left())
      {
        if (space_right3())
        {
          if (!moveright())
            return;
          xi++;
        }
        else return;
      }     
      else if (!space_right())
      {
        if (space_left3())
        {
          if (!moveleft())
            return;
          if (!moveleft())
            return;          
          xi--;
          xi--;        
        }
        else
          return;
      }
      else if (!space_right2())
      {
        if (space_left2())
        {
          if (!moveleft())
            return;          
          xi--;      
        }
        else
          return;
      }   
      block[xi][yi]=0;
      block[xi][yi+2]=0;
      block[xi][yi+3]=0;      
      block[xi-1][yi+1]=1;
      block[xi+1][yi+1]=1;
      block[xi+2][yi+1]=1;      
      blockrotation = 1;
    }
    else
    {
      block[xi][yi]=0;
      block[xi+2][yi]=0;
      block[xi+3][yi]=0;
      block[xi+1][yi-1]=1;
      block[xi+1][yi+1]=1;
      block[xi+1][yi+2]=1;
      blockrotation = 0;
    }    
  }
  //offset to mid
  xi ++;  
  yi ++;  
  if (blocktype == 1)
  {
    if (blockrotation == 0)
    {
      block[xi-1][yi-1] = 0;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 0;
      block[xi][yi-1] = 1;
      block[xi+1][yi-1] = 1;
      block[xi][yi+1] = 1;      
      blockrotation = 1;
    }
    else if (blockrotation == 1)
    {
      if (!space_left())
      {
        if (!moveright())
          return;
        xi++;
      }        
      xi--;
      block[xi][yi-1] = 0;
      block[xi+1][yi-1] = 0;
      block[xi][yi+1] = 0;      
      block[xi-1][yi] = 1;
      block[xi+1][yi] = 1;
      block[xi+1][yi+1] = 1;      
      blockrotation = 2;      
    }
    else if (blockrotation == 2)
    {
      yi --;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 0;
      block[xi+1][yi+1] = 0;      
      block[xi][yi-1] = 1;
      block[xi][yi+1] = 1;
      block[xi-1][yi+1] = 1;      
      blockrotation = 3;            
    }
    else
    {
      if (!space_right())
      {
        if (!moveleft())
          return;
        xi--;
      }
      block[xi][yi-1] = 0;
      block[xi][yi+1] = 0;
      block[xi-1][yi+1] = 0;        
      block[xi-1][yi-1] = 1;
      block[xi-1][yi] = 1;
      block[xi+1][yi] = 1;
      blockrotation = 0;          
    }  
  }
  if (blocktype == 2)
  {
    if (blockrotation == 0)
    {
      block[xi+1][yi-1] = 0;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 0;
      block[xi][yi-1] = 1;
      block[xi+1][yi+1] = 1;
      block[xi][yi+1] = 1;      
      blockrotation = 1;
    }
    else if (blockrotation == 1)
    {
      if (!space_left())
      {
        if (!moveright())
          return;
        xi++;
      }              
      xi--;
      block[xi][yi-1] = 0;
      block[xi+1][yi+1] = 0;
      block[xi][yi+1] = 0;      
      block[xi-1][yi] = 1;
      block[xi+1][yi] = 1;
      block[xi-1][yi+1] = 1;      
      blockrotation = 2;      
    }
    else if (blockrotation == 2)
    {
      yi --;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 0;
      block[xi-1][yi+1] = 0;      
      block[xi][yi-1] = 1;
      block[xi][yi+1] = 1;
      block[xi-1][yi-1] = 1;      
      blockrotation = 3;            
    }
    else
    {
      if (!space_right())
      {
        if (!moveleft())
          return;
        xi--;
      }      
      block[xi][yi-1] = 0;
      block[xi][yi+1] = 0;
      block[xi-1][yi-1] = 0;        
      block[xi+1][yi-1] = 1;
      block[xi-1][yi] = 1;
      block[xi+1][yi] = 1;
      blockrotation = 0;          
    }  
  }
  if (blocktype == 4)
  {
    if (blockrotation == 0)
    {
      block[xi+1][yi-1] = 0;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 1;
      block[xi+1][yi+1] = 1;      
      blockrotation = 1;
    }
    else
    {
      if (!space_left())
      {
        if (!moveright())
          return;
        xi++;
      }              
      xi--;
      block[xi+1][yi] = 0;
      block[xi+1][yi+1] = 0;      
      block[xi-1][yi] = 1;
      block[xi+1][yi-1] = 1;
      blockrotation = 0;          
    }  
  }  
  if (blocktype == 5)
  {
    if (blockrotation == 0)
    {
      block[xi][yi-1] = 0;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 0;
      block[xi][yi-1] = 1;
      block[xi+1][yi] = 1;
      block[xi][yi+1] = 1;      
      blockrotation = 1;
    }
    else if (blockrotation == 1)
    {
      if (!space_left())
      {
        if (!moveright())
          return;
        xi++;
      }              
      xi--;
      block[xi][yi-1] = 0;
      block[xi+1][yi] = 0;
      block[xi][yi+1] = 0;
      block[xi-1][yi] = 1;
      block[xi+1][yi] = 1;
      block[xi][yi+1] = 1;
      blockrotation = 2;      
    }
    else if (blockrotation == 2)
    {
      yi --;
      block[xi-1][yi] = 0;
      block[xi+1][yi] = 0;
      block[xi][yi+1] = 0;     
      block[xi][yi-1] = 1;
      block[xi-1][yi] = 1;
      block[xi][yi+1] = 1;      
      blockrotation = 3;            
    }
    else
    {
      if (!space_right())
      {
        if (!moveleft())
          return;
        xi--;
      }      
      block[xi][yi-1] = 0;
      block[xi-1][yi] = 0;
      block[xi][yi+1] = 0;      
      block[xi][yi-1] = 1;
      block[xi-1][yi] = 1;
      block[xi+1][yi] = 1;
      blockrotation = 0;          
    }  
  }
  if (blocktype == 6)
  {
    if (blockrotation == 0)
    {
      block[xi-1][yi-1] = 0;
      block[xi][yi-1] = 0;
      block[xi+1][yi-1] = 1;
      block[xi][yi+1] = 1;      
      blockrotation = 1;
    }
    else
    {
      if (!space_left())
      {
        if (!moveright())
          return;
        xi++;
      }              
      xi--;
      block[xi+1][yi-1] = 0;
      block[xi][yi+1] = 0;      
      block[xi-1][yi-1] = 1;
      block[xi][yi-1] = 1;
      blockrotation = 0;          
    }  
  }  
  //if rotating made block and pile overlap, push rows up
  while (!check_overlap())
  {
    for (i=0;i<Block_column_size+2;i++)
    {
      for (j=0;j<Block_row_size;j++)
      {
        block[j][i] = block[j][i+1];
      }
    }
  }
  updateLED();
}

//*************************************************************************************************//
bool space_left(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (j == 0)
          return false;
        if (pile[j-1][i])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
bool space_left2(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (j == 0 || j == 1)
          return false;
        if (pile[j-1][i] | pile[j-2][i])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
bool space_left3(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (j == 0 || j == 1 ||j == 2 )
          return false;
        if (pile[j-1][i] | pile[j-2][i]|pile[j-3][i])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
bool space_right(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (j == Block_row_size-1)
          return false;
        if (pile[j+1][i])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
bool space_right2(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (j == 9 || j == 8)
          return false;
        if (pile[j+1][i] |pile[j+2][i])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
bool space_right3(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (j == 9||j == 8||j == 7)
          return false;
        if (pile[j+1][i] |pile[j+2][i] | pile[j+3][i])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
bool space_below(){
  int i;
  int j;  
  for (i=Block_column_size-1;i>=0;i--)
  {
    for (j=0;j<Block_row_size;j++)
    {
      if (block[j][i])
      {
        if (i == Block_column_size-1)
          return false;
        if (pile[j][i+1])
        {
          return false;
        }      
      }        
    }
  }
  return true;
}

//*************************************************************************************************//
void drop_new_block(){
  if (run_status == 0)
  {
    blocktype = random(7); //  Generate First Block
  }
  else
  {
    blocktype = next_piece;  //Drop Piece from previous generation
  }
  blockcolor = blocktype + 1;
  if (blocktype == 0)
    // 0
    // 0
    // 0
    // 0
  {
    block[4][0]=1;
    block[4][1]=1;
    block[4][2]=1;
    block[4][3]=1;
  }
  if (blocktype == 1)
    // 0
    // 0 0 0
  {
    block[3][0]=1;
    block[3][1]=1;
    block[4][1]=1;
    block[5][1]=1;
  }
  if (blocktype == 2)
    //     0
    // 0 0 0
  {
    block[5][0]=1;
    block[3][1]=1;
    block[4][1]=1;
    block[5][1]=1;
  }
  if (blocktype == 3)
    // 0 0
    // 0 0
  {
    block[4][0]=1;
    block[4][1]=1;
    block[5][0]=1;
    block[5][1]=1;
  }    
  if (blocktype == 4)
    //   0 0
    // 0 0
  {
    block[5][0]=1;
    block[6][0]=1;
    block[4][1]=1;
    block[5][1]=1;
  }    
  if (blocktype == 5)
    //   0
    // 0 0 0
  {
    block[5][0]=1;
    block[4][1]=1;
    block[5][1]=1;
    block[6][1]=1;
  }        
  if (blocktype == 6)
    // 0 0
    //   0 0
  {
    block[4][0]=1;
    block[5][0]=1;
    block[5][1]=1;
    block[6][1]=1;
  }    
  blockrotation = 0;
}

//*************************************************************************************************//
bool check_overlap(){
  int i;
  int j;  
  for (i=0;i<Block_column_size;i++)
  {
    for (j=0;j<9;j++)
    {
      if (block[j][i])
      {
        if (pile[j][i])
          return false;
      }        
    }
  }
  for (i=Block_column_size;i<Block_column_size+2;i++)
  {
    for (j=0;j<Block_row_size-1;j++)
    {
      if (block[j][i])
      {
        return false;
      }        
    }
  }  
  return true;
}

//*************************************************************************************************//
void check_gameover(){
  int i;
  for(i=0;i<Block_row_size;i++)
  {
    if (pile[i][0])
    {
      gameover_flag = 1;
    }
  }
  if (gameover_flag == 1)
  {
    gameover();
  } 
}

//*************************************************************************************************//
void updateLED(){
  int i;
  int j;
  for (i=0;i<Block_row_size;i++)
  {
    for (j=0;j<Block_column_size;j++)
    {
      disp[i][j] = (block[i][j] | pile[i][j]);
    }
  }
}

//*************************************************************************************************//
void LEDRefresh(){
  clear_mini_arrays();
  int i;
  int k;
  for(i=0;i<Block_column_size/2;i++)
  {      
    word upper = 0;
    int b;
    for(b = 0;b<Block_row_size;b++)
    {
      upper <<= 1;
      if (disp[b][i]) 
      {
        upper |= 1;
      }
    }
    send_data(1, i, upper);
    word lower = 0;
    for(b = 0;b<Block_row_size;b++)
    {
      lower <<= 1;
      if (disp[b][i+Block_column_size/2])
      {
        lower |= 1;
      }   
    }
    send_data(0, i, lower);
    delay(1);
    upper = 0;
    lower = 0;
  }   
}

//*************************************************************************************************//
void send_data(unsigned int array, unsigned int y_axis, unsigned int data){
  if (array == 0)
  { 
    if ( (data/512) == 1)
    {
      mini_lower[y_axis][9] = 1;
      data = data - 512;
    }
    else {
      mini_lower[y_axis][9] = 0;
    }
    if ( (data/256) == 1)
    {
      mini_lower[y_axis][8] = 1;
      data = data - 256;
    }
    else {
      mini_lower[y_axis][8] = 0;
    }
    if ( (data/128) == 1)
    {
      mini_lower[y_axis][7] = 1;
      data = data - 128;
    }
    else {
      mini_lower[y_axis][7] = 0;
    }
    if ( (data/64) == 1)
    {
      mini_lower[y_axis][6] = 1;
      data = data - 64;
    }
    else {
      mini_lower[y_axis][6] = 0;
    }
    if ( (data/32) == 1)
    {
      mini_lower[y_axis][5] = 1;
      data = data - 32;
    }
    else {
      mini_lower[y_axis][5] = 0;
    }
    if ( (data/16) == 1)
    {
      mini_lower[y_axis][4] = 1;
      data = data - 16;
    }
    else {
      mini_lower[y_axis][4] = 0;
    }
    if ( (data/8) == 1)
    {
      mini_lower[y_axis][3] = 1;
      data = data - 8;
    }
    else {
      mini_lower[y_axis][3] = 0;
    }
    if ( (data/4) == 1)
    {
      mini_lower[y_axis][2] = 1;
      data = data - 4;
    }
    else {
      mini_lower[y_axis][2] = 0;
    }
    if ( (data/2) == 1)
    {
      mini_lower[y_axis][1] = 1;
      data = data - 2;
    }
    else {
      mini_lower[y_axis][1] = 0;
    }
    if ( (data/1) == 1)
    {
      mini_lower[y_axis][0] = 1;
    }
    else {
      mini_lower[y_axis][0] = 0;
    }
  }
  else 
  {
    if ( (data/512) == 1)
    {
      mini_upper[y_axis][9] = 1;
      data = data - 512;
    }
    else {
      mini_lower[y_axis][9] = 0;
    }
    if ( (data/256) == 1)
    {
      mini_upper[y_axis][8] = 1;
      data = data - 256;
    }
    else {
      mini_lower[y_axis][8] = 0;
    }
    if ( (data/128) == 1)
    {
      mini_upper[y_axis][7] = 1;
      data = data - 128;
    }
    else {
      mini_lower[y_axis][7] = 0;
    }
    if ( (data/64) == 1)
    {
      mini_upper[y_axis][6] = 1;
      data = data - 64;
    }
    else {
      mini_lower[y_axis][6] = 0;
    }
    if ( (data/32) == 1)
    {
      mini_upper[y_axis][5] = 1;
      data = data - 32;
    }
    else {
      mini_lower[y_axis][5] = 0;
    }
    if ( (data/16) == 1)
    {
      mini_upper[y_axis][4] = 1;
      data = data - 16;
    }
    else {
      mini_lower[y_axis][4] = 0;
    }
    if ( (data/8) == 1)
    {
      mini_upper[y_axis][3] = 1;
      data = data - 8;
    }
    else {
      mini_lower[y_axis][3] = 0;
    }
    if ( (data/4) == 1)
    {
      mini_upper[y_axis][2] = 1;
      data = data - 4;
    }
    else {
      mini_lower[y_axis][2] = 0;
    }
    if ( (data/2) == 1)
    {
      mini_upper[y_axis][1] = 1;
      data = data - 2;
    }
    else {
      mini_lower[y_axis][1] = 0;
    }
    if ( (data/1) == 1)
    {
      mini_upper[y_axis][0] = 1;
    }
    else {
      mini_lower[y_axis][0] = 0;
    }
  }
}

//*************************************************************************************************//
void load_datamap(){
  
  for ( int y = 0; y<Block_column_size; y++)
  {
    for ( int x = 0; x < Block_row_size; x++)
    {
      used_full_array[y][x] = full_array[y][x];
    }
  }   
  unsigned int j = 0;
  for (y_axis = 0; y_axis<((Block_column_size/2)+1); y_axis++)
  {
    for (x_axis = 0; x_axis < Block_row_size; x_axis++)
    {
      full_array[y_axis][x_axis] = mini_upper[j][x_axis];
    }
    j++;
  }
  unsigned int i = 0;  
  for (y_axis = Block_column_size/2; y_axis<Block_column_size+1; y_axis++)
  {
    for (x_axis = 0; x_axis < Block_row_size; x_axis++)
    {
      full_array[y_axis][x_axis] = mini_lower[i][x_axis];
    }
    i++;
  }
  int Q = 0;
  for (int x = Block_row_size; x > 0; x--)
  {
    for ( int y = 0; y<Block_column_size; y++)
    {
      if (  ( full_array[y][x-1] == 1 ) && ( used_full_array[y][x-1] == 0 ) && ( gameover_flag == 0 )  )
      {  
        temp_Block_datamap[Q] = blockcolor;  
        Q++;
      }
      else if (  ( full_array[y][x-1] == 1 ) && ( used_full_array[y][x-1] == 1 ) && ( gameover_flag == 0 )  )
      {                      
        Q++;
      }
      else if (( full_array[y][x-1] == 1) && (gameover_flag == 1))
      {
        temp_Block_datamap[Q] = random(1,9);
        Q++;   
      }
      else
      {
        temp_Block_datamap[Q] = 0;
        Q++;
      }
    }
  }  
}

//*************************************************************************************************//
void select_strip_index(){
  if (current_Block_strip % 2 == 0)
  {
    unsigned int w = (((current_Block_strip + 1) * Block_column_size)- 1);
    for (unsigned int x = 0; x < Block_column_size; x++)
    {
      LED_index[x] = w;
      w--;
    }
  }
  else
  {
    unsigned int w = current_Block_strip * Block_column_size;
    for (unsigned int x = 0; x < Block_column_size; x++)
    {
      LED_index[x] = w;
      w++;
    }
  }
}

//*************************************************************************************************//
void send_strip_data(unsigned int status){
  noInterrupts();
  switch (status)
  {
  case 0:  // Clear LED Display
    for(int x = 0; x < LED_total_count; x++)
    {
      strip.HSVsetLEDColor(x, 0,0,0);
    }  
    break;
  case 1:  // Game LED Display
    for(int x = 0; x < LED_total_count; x++)
    {
      strip.HSVsetLEDColor(x, disp_colors[output_data_map[x]][0] /* Hue */ , disp_colors[output_data_map[x]][1] /* Sat */, disp_colors[output_data_map[x]][2] /* Val */);
    }  
    break;
  case 2:  // Logo Display
    for(int x = 0; x < LED_total_count; x++)
    {
      strip.HSVsetLEDColor(x, logo_colors[Logo_Datamap[x]][0], logo_colors[Logo_Datamap[x]][1], logo_colors[Logo_Datamap[x]][2]);
    } 
  }
  strip.refreshLEDs();
  interrupts();
}

//*************************************************************************************************//
void clear_mini_arrays(){
  for ( int y = 0; y<((Block_column_size/2)+1); y++)
  {
    for ( int x = 0; x < Block_row_size; x++)
    {
      mini_upper[y][x] = 0;
      mini_lower[y][x] = 0;
    }
  }
}

//*************************************************************************************************//
void create_matrix_map(){
  for( unsigned int a = 0; a < number_of_Block_LEDs; a++)
  {
    Block_datamap[a] = 0;
  }
  unsigned int LED_Count = 0;
  for (current_Block_strip = 0; current_Block_strip < number_of_Block_strips; current_Block_strip++)
  {  
    select_strip_index();
    for (unsigned int v = 0; v < Block_column_size; v++)
    {
      Block_datamap[LED_Count] = temp_Block_datamap[LED_index[v]];
      LED_Count++;
    }  
  }
}

//*************************************************************************************************//
void Blockmatrix_update(){
  LEDRefresh();
  load_datamap(); 
  create_matrix_map();
  Block_map_update();
  send_strip_data(1);
}

//*************************************************************************************************//
void clear_all_arrays(){
  x_axis = 0;
  y_axis = 0; 
  number_of_Block_strips = number_of_Block_LEDs /Block_column_size;
  current_Block_strip = 0;
  for (int a = 0; a < Block_column_size; a++)
  {
    for (int b = 0; b < Block_row_size; b++)
    {
      full_array[a][b] = 0;
    }
  }
  for ( int y = 0; y<((Block_column_size/2)+1); y++)
  {
    for ( int x = 0; x < Block_row_size; x++)
    {
      mini_upper[y][x] = 0;
      mini_lower[y][x] = 0;
    }
  }  
  for (int a = 0; a < Block_row_size; a++)
  {
    for (int b = 0; b < Block_column_size+2; b++)
    {
      block[a][b] = 0;
    }
  }  
  for (int a = 0; a < Block_row_size; a++)
  {
    for (int b = 0; b < Block_column_size; b++)
    {
      pile[a][b] = 0;
    }
  }
  for (int a = 0; a < Block_row_size; a++)
  {
    for (int b = 0; b < Block_column_size; b++)
    {
      disp[a][b] = 0;
    }
  }
  for( unsigned int a = 0; a < number_of_Block_LEDs; a++)
  {
    Block_datamap[a] = 0;
    temp_Block_datamap[a] = 0;
  }
  for( unsigned int a = 0; a < Block_column_size; a++)
  {
    LED_index[a] = 0;
  }
}

//*************************************************************************************************//
void gameover(){
  unsigned int i = 0;
  unsigned int j = 0;
  for (i=0;i<Block_row_size;i++)
  {
    for (j=0;j<Block_column_size;j++)
    {
      if (j%2)
      {
        disp[i][j]=1;
      }
      else
      {
        disp[(Block_row_size-1)-i][j]=1;
      }
    }
    delay(20);
    LEDRefresh();
    load_datamap(); 
    create_matrix_map();
    Block_map_update();
    send_strip_data(1);
  }  
  for (i=0;i<Block_row_size;i++)
  {
    for (j=0;j<Block_column_size;j++)
    {
      if (j%2)
      {
        disp[i][j]=0;
      }
      else
      {
        disp[(Block_row_size-1)-i][j]=0;
      }
    }
    delay(20);
    LEDRefresh();
    load_datamap(); 
    create_matrix_map();
    Block_map_update();
    send_strip_data(1);
  }
  next_piece = 7;
  generate_next_piece_gfx();
  send_strip_data(1);
  next_piece = 0;
  delay(5000);
  run_status = 0;
  clear_all_arrays();
  send_strip_data(0);
  total_lines_cleared = 0;
  difficulty = 100;
  drop_difficulty_count = 0;
  clear_master_data_map();
}

//*************************************************************************************************//
void clear_lines(){
  int i;
  int j;
  int count=0;
  for(i=Block_column_size-1;i>=0;i--)
  {
    count=0;
    for (j=0;j<Block_row_size;j++)
    {
      if (pile[j][i])
      {
        count ++;
      }
    }
    // CLEARS THE LINE   
    if (count == Block_row_size)
    {
      row = i;
      int z = 0;     
      for ( int x = 0; x < Block_row_size; x++)
      {
        for ( int y = 0; y < Block_column_size; y++)
        {
          color_array[y][x] = temp_Block_datamap[z];
          z++;      
        }
      }
      for (j=0;j<Block_row_size;j++)
      {
        pile[j][i] = 0;
        int index = (j*Block_column_size)+ row;
        cleared_index[j] = index;
        temp_Block_datamap[index] = 8;// set white color
        create_matrix_map();
        Block_map_update();
        send_strip_data(1);
        delay(35);
      }
      delay(350);
      total_lines_cleared++;
      if (total_lines_cleared == 5)
      {difficulty = 100;}
      if (total_lines_cleared == 10)
      {difficulty = 75;}
      if (total_lines_cleared == 13)
      {difficulty = 50;}
      if (total_lines_cleared == 15)
      {difficulty = 25;}
      score_char_update();
      for( int x = 0; x < Block_row_size; x++ )
      {
        for ( int y = row + 1; y > 0; y-- )
        {
          color_array[y-1][x] = color_array[y-2][x];
          color_array[0][x] = 0;
        }
      }
      z = 0;
      for (int x = 0; x < Block_row_size; x++)
      {
        for (int y = 0; y < Block_column_size; y++)
        {
          temp_Block_datamap[z] = color_array[y][x];
          z++;
        }
      }
      // DROPS THE PILE   
      int k;
      for(k=i;k>0;k--)
      {
        for (j=0;j<Block_row_size;j++)
        {
          pile[j][k] = pile[j][k-1];
        }
      }
      for (j=0;j<Block_row_size;j++)
      {
        pile[j][0] = 0;
      }
      updateLED();
      i++;  
    }
  }
}

//*************************************************************************************************//
void score_char_update(){
  int left_char = total_lines_cleared / 10;
  int right_char = total_lines_cleared % 10;
  switch(left_char){
case 0:	
output_data_map[464]	=	num_char0[0][0];
output_data_map[463]	=	num_char0[0][1];
output_data_map[462]	=	num_char0[0][2];
output_data_map[461]	=	num_char0[0][3];
output_data_map[460]	=	num_char0[0][4];
output_data_map[459]	=	num_char0[0][5];
output_data_map[458]	=	num_char0[0][6];
output_data_map[457]	=	num_char0[0][7];
output_data_map[502]	=	num_char0[1][0];
output_data_map[501]	=	num_char0[1][1];
output_data_map[500]	=	num_char0[1][2];
output_data_map[499]	=	num_char0[1][3];
output_data_map[498]	=	num_char0[1][4];
output_data_map[497]	=	num_char0[1][5];
output_data_map[496]	=	num_char0[1][6];
output_data_map[495]	=	num_char0[1][7];
output_data_map[524]	=	num_char0[2][0];
output_data_map[523]	=	num_char0[2][1];
output_data_map[522]	=	num_char0[2][2];
output_data_map[521]	=	num_char0[2][3];
output_data_map[520]	=	num_char0[2][4];
output_data_map[519]	=	num_char0[2][5];
output_data_map[518]	=	num_char0[2][6];
output_data_map[517]	=	num_char0[2][7];
output_data_map[562]	=	num_char0[3][0];
output_data_map[561]	=	num_char0[3][1];
output_data_map[560]	=	num_char0[3][2];
output_data_map[559]	=	num_char0[3][3];
output_data_map[558]	=	num_char0[3][4];
output_data_map[557]	=	num_char0[3][5];
output_data_map[556]	=	num_char0[3][6];
output_data_map[555]	=	num_char0[3][7];
output_data_map[584]	=	num_char0[4][0];
output_data_map[583]	=	num_char0[4][1];
output_data_map[582]	=	num_char0[4][2];
output_data_map[581]	=	num_char0[4][3];
output_data_map[580]	=	num_char0[4][4];
output_data_map[579]	=	num_char0[4][5];
output_data_map[578]	=	num_char0[4][6];
output_data_map[577]	=	num_char0[4][7];
break;		
case	1:	
output_data_map[464]	=	num_char1[0][0];
output_data_map[463]	=	num_char1[0][1];
output_data_map[462]	=	num_char1[0][2];
output_data_map[461]	=	num_char1[0][3];
output_data_map[460]	=	num_char1[0][4];
output_data_map[459]	=	num_char1[0][5];
output_data_map[458]	=	num_char1[0][6];
output_data_map[457]	=	num_char1[0][7];
output_data_map[502]	=	num_char1[1][0];
output_data_map[501]	=	num_char1[1][1];
output_data_map[500]	=	num_char1[1][2];
output_data_map[499]	=	num_char1[1][3];
output_data_map[498]	=	num_char1[1][4];
output_data_map[497]	=	num_char1[1][5];
output_data_map[496]	=	num_char1[1][6];
output_data_map[495]	=	num_char1[1][7];
output_data_map[524]	=	num_char1[2][0];
output_data_map[523]	=	num_char1[2][1];
output_data_map[522]	=	num_char1[2][2];
output_data_map[521]	=	num_char1[2][3];
output_data_map[520]	=	num_char1[2][4];
output_data_map[519]	=	num_char1[2][5];
output_data_map[518]	=	num_char1[2][6];
output_data_map[517]	=	num_char1[2][7];
output_data_map[562]	=	num_char1[3][0];
output_data_map[561]	=	num_char1[3][1];
output_data_map[560]	=	num_char1[3][2];
output_data_map[559]	=	num_char1[3][3];
output_data_map[558]	=	num_char1[3][4];
output_data_map[557]	=	num_char1[3][5];
output_data_map[556]	=	num_char1[3][6];
output_data_map[555]	=	num_char1[3][7];
output_data_map[584]	=	num_char1[4][0];
output_data_map[583]	=	num_char1[4][1];
output_data_map[582]	=	num_char1[4][2];
output_data_map[581]	=	num_char1[4][3];
output_data_map[580]	=	num_char1[4][4];
output_data_map[579]	=	num_char1[4][5];
output_data_map[578]	=	num_char1[4][6];
output_data_map[577]	=	num_char1[4][7];
break;		
case	2:	
output_data_map[464]	=	num_char2[0][0];
output_data_map[463]	=	num_char2[0][1];
output_data_map[462]	=	num_char2[0][2];
output_data_map[461]	=	num_char2[0][3];
output_data_map[460]	=	num_char2[0][4];
output_data_map[459]	=	num_char2[0][5];
output_data_map[458]	=	num_char2[0][6];
output_data_map[457]	=	num_char2[0][7];
output_data_map[502]	=	num_char2[1][0];
output_data_map[501]	=	num_char2[1][1];
output_data_map[500]	=	num_char2[1][2];
output_data_map[499]	=	num_char2[1][3];
output_data_map[498]	=	num_char2[1][4];
output_data_map[497]	=	num_char2[1][5];
output_data_map[496]	=	num_char2[1][6];
output_data_map[495]	=	num_char2[1][7];
output_data_map[524]	=	num_char2[2][0];
output_data_map[523]	=	num_char2[2][1];
output_data_map[522]	=	num_char2[2][2];
output_data_map[521]	=	num_char2[2][3];
output_data_map[520]	=	num_char2[2][4];
output_data_map[519]	=	num_char2[2][5];
output_data_map[518]	=	num_char2[2][6];
output_data_map[517]	=	num_char2[2][7];
output_data_map[562]	=	num_char2[3][0];
output_data_map[561]	=	num_char2[3][1];
output_data_map[560]	=	num_char2[3][2];
output_data_map[559]	=	num_char2[3][3];
output_data_map[558]	=	num_char2[3][4];
output_data_map[557]	=	num_char2[3][5];
output_data_map[556]	=	num_char2[3][6];
output_data_map[555]	=	num_char2[3][7];
output_data_map[584]	=	num_char2[4][0];
output_data_map[583]	=	num_char2[4][1];
output_data_map[582]	=	num_char2[4][2];
output_data_map[581]	=	num_char2[4][3];
output_data_map[580]	=	num_char2[4][4];
output_data_map[579]	=	num_char2[4][5];
output_data_map[578]	=	num_char2[4][6];
output_data_map[577]	=	num_char2[4][7];
break;		
case	3:	
output_data_map[464]	=	num_char3[0][0];
output_data_map[463]	=	num_char3[0][1];
output_data_map[462]	=	num_char3[0][2];
output_data_map[461]	=	num_char3[0][3];
output_data_map[460]	=	num_char3[0][4];
output_data_map[459]	=	num_char3[0][5];
output_data_map[458]	=	num_char3[0][6];
output_data_map[457]	=	num_char3[0][7];
output_data_map[502]	=	num_char3[1][0];
output_data_map[501]	=	num_char3[1][1];
output_data_map[500]	=	num_char3[1][2];
output_data_map[499]	=	num_char3[1][3];
output_data_map[498]	=	num_char3[1][4];
output_data_map[497]	=	num_char3[1][5];
output_data_map[496]	=	num_char3[1][6];
output_data_map[495]	=	num_char3[1][7];
output_data_map[524]	=	num_char3[2][0];
output_data_map[523]	=	num_char3[2][1];
output_data_map[522]	=	num_char3[2][2];
output_data_map[521]	=	num_char3[2][3];
output_data_map[520]	=	num_char3[2][4];
output_data_map[519]	=	num_char3[2][5];
output_data_map[518]	=	num_char3[2][6];
output_data_map[517]	=	num_char3[2][7];
output_data_map[562]	=	num_char3[3][0];
output_data_map[561]	=	num_char3[3][1];
output_data_map[560]	=	num_char3[3][2];
output_data_map[559]	=	num_char3[3][3];
output_data_map[558]	=	num_char3[3][4];
output_data_map[557]	=	num_char3[3][5];
output_data_map[556]	=	num_char3[3][6];
output_data_map[555]	=	num_char3[3][7];
output_data_map[584]	=	num_char3[4][0];
output_data_map[583]	=	num_char3[4][1];
output_data_map[582]	=	num_char3[4][2];
output_data_map[581]	=	num_char3[4][3];
output_data_map[580]	=	num_char3[4][4];
output_data_map[579]	=	num_char3[4][5];
output_data_map[578]	=	num_char3[4][6];
output_data_map[577]	=	num_char3[4][7];
break;		
case	4:	
output_data_map[464]	=	num_char4[0][0];
output_data_map[463]	=	num_char4[0][1];
output_data_map[462]	=	num_char4[0][2];
output_data_map[461]	=	num_char4[0][3];
output_data_map[460]	=	num_char4[0][4];
output_data_map[459]	=	num_char4[0][5];
output_data_map[458]	=	num_char4[0][6];
output_data_map[457]	=	num_char4[0][7];
output_data_map[502]	=	num_char4[1][0];
output_data_map[501]	=	num_char4[1][1];
output_data_map[500]	=	num_char4[1][2];
output_data_map[499]	=	num_char4[1][3];
output_data_map[498]	=	num_char4[1][4];
output_data_map[497]	=	num_char4[1][5];
output_data_map[496]	=	num_char4[1][6];
output_data_map[495]	=	num_char4[1][7];
output_data_map[524]	=	num_char4[2][0];
output_data_map[523]	=	num_char4[2][1];
output_data_map[522]	=	num_char4[2][2];
output_data_map[521]	=	num_char4[2][3];
output_data_map[520]	=	num_char4[2][4];
output_data_map[519]	=	num_char4[2][5];
output_data_map[518]	=	num_char4[2][6];
output_data_map[517]	=	num_char4[2][7];
output_data_map[562]	=	num_char4[3][0];
output_data_map[561]	=	num_char4[3][1];
output_data_map[560]	=	num_char4[3][2];
output_data_map[559]	=	num_char4[3][3];
output_data_map[558]	=	num_char4[3][4];
output_data_map[557]	=	num_char4[3][5];
output_data_map[556]	=	num_char4[3][6];
output_data_map[555]	=	num_char4[3][7];
output_data_map[584]	=	num_char4[4][0];
output_data_map[583]	=	num_char4[4][1];
output_data_map[582]	=	num_char4[4][2];
output_data_map[581]	=	num_char4[4][3];
output_data_map[580]	=	num_char4[4][4];
output_data_map[579]	=	num_char4[4][5];
output_data_map[578]	=	num_char4[4][6];
output_data_map[577]	=	num_char4[4][7];
break;		
case	5:	
output_data_map[464]	=	num_char5[0][0];
output_data_map[463]	=	num_char5[0][1];
output_data_map[462]	=	num_char5[0][2];
output_data_map[461]	=	num_char5[0][3];
output_data_map[460]	=	num_char5[0][4];
output_data_map[459]	=	num_char5[0][5];
output_data_map[458]	=	num_char5[0][6];
output_data_map[457]	=	num_char5[0][7];
output_data_map[502]	=	num_char5[1][0];
output_data_map[501]	=	num_char5[1][1];
output_data_map[500]	=	num_char5[1][2];
output_data_map[499]	=	num_char5[1][3];
output_data_map[498]	=	num_char5[1][4];
output_data_map[497]	=	num_char5[1][5];
output_data_map[496]	=	num_char5[1][6];
output_data_map[495]	=	num_char5[1][7];
output_data_map[524]	=	num_char5[2][0];
output_data_map[523]	=	num_char5[2][1];
output_data_map[522]	=	num_char5[2][2];
output_data_map[521]	=	num_char5[2][3];
output_data_map[520]	=	num_char5[2][4];
output_data_map[519]	=	num_char5[2][5];
output_data_map[518]	=	num_char5[2][6];
output_data_map[517]	=	num_char5[2][7];
output_data_map[562]	=	num_char5[3][0];
output_data_map[561]	=	num_char5[3][1];
output_data_map[560]	=	num_char5[3][2];
output_data_map[559]	=	num_char5[3][3];
output_data_map[558]	=	num_char5[3][4];
output_data_map[557]	=	num_char5[3][5];
output_data_map[556]	=	num_char5[3][6];
output_data_map[555]	=	num_char5[3][7];
output_data_map[584]	=	num_char5[4][0];
output_data_map[583]	=	num_char5[4][1];
output_data_map[582]	=	num_char5[4][2];
output_data_map[581]	=	num_char5[4][3];
output_data_map[580]	=	num_char5[4][4];
output_data_map[579]	=	num_char5[4][5];
output_data_map[578]	=	num_char5[4][6];
output_data_map[577]	=	num_char5[4][7];
break;		
case	6:	
output_data_map[464]	=	num_char6[0][0];
output_data_map[463]	=	num_char6[0][1];
output_data_map[462]	=	num_char6[0][2];
output_data_map[461]	=	num_char6[0][3];
output_data_map[460]	=	num_char6[0][4];
output_data_map[459]	=	num_char6[0][5];
output_data_map[458]	=	num_char6[0][6];
output_data_map[457]	=	num_char6[0][7];
output_data_map[502]	=	num_char6[1][0];
output_data_map[501]	=	num_char6[1][1];
output_data_map[500]	=	num_char6[1][2];
output_data_map[499]	=	num_char6[1][3];
output_data_map[498]	=	num_char6[1][4];
output_data_map[497]	=	num_char6[1][5];
output_data_map[496]	=	num_char6[1][6];
output_data_map[495]	=	num_char6[1][7];
output_data_map[524]	=	num_char6[2][0];
output_data_map[523]	=	num_char6[2][1];
output_data_map[522]	=	num_char6[2][2];
output_data_map[521]	=	num_char6[2][3];
output_data_map[520]	=	num_char6[2][4];
output_data_map[519]	=	num_char6[2][5];
output_data_map[518]	=	num_char6[2][6];
output_data_map[517]	=	num_char6[2][7];
output_data_map[562]	=	num_char6[3][0];
output_data_map[561]	=	num_char6[3][1];
output_data_map[560]	=	num_char6[3][2];
output_data_map[559]	=	num_char6[3][3];
output_data_map[558]	=	num_char6[3][4];
output_data_map[557]	=	num_char6[3][5];
output_data_map[556]	=	num_char6[3][6];
output_data_map[555]	=	num_char6[3][7];
output_data_map[584]	=	num_char6[4][0];
output_data_map[583]	=	num_char6[4][1];
output_data_map[582]	=	num_char6[4][2];
output_data_map[581]	=	num_char6[4][3];
output_data_map[580]	=	num_char6[4][4];
output_data_map[579]	=	num_char6[4][5];
output_data_map[578]	=	num_char6[4][6];
output_data_map[577]	=	num_char6[4][7];
break;		
case	7:	
output_data_map[464]	=	num_char7[0][0];
output_data_map[463]	=	num_char7[0][1];
output_data_map[462]	=	num_char7[0][2];
output_data_map[461]	=	num_char7[0][3];
output_data_map[460]	=	num_char7[0][4];
output_data_map[459]	=	num_char7[0][5];
output_data_map[458]	=	num_char7[0][6];
output_data_map[457]	=	num_char7[0][7];
output_data_map[502]	=	num_char7[1][0];
output_data_map[501]	=	num_char7[1][1];
output_data_map[500]	=	num_char7[1][2];
output_data_map[499]	=	num_char7[1][3];
output_data_map[498]	=	num_char7[1][4];
output_data_map[497]	=	num_char7[1][5];
output_data_map[496]	=	num_char7[1][6];
output_data_map[495]	=	num_char7[1][7];
output_data_map[524]	=	num_char7[2][0];
output_data_map[523]	=	num_char7[2][1];
output_data_map[522]	=	num_char7[2][2];
output_data_map[521]	=	num_char7[2][3];
output_data_map[520]	=	num_char7[2][4];
output_data_map[519]	=	num_char7[2][5];
output_data_map[518]	=	num_char7[2][6];
output_data_map[517]	=	num_char7[2][7];
output_data_map[562]	=	num_char7[3][0];
output_data_map[561]	=	num_char7[3][1];
output_data_map[560]	=	num_char7[3][2];
output_data_map[559]	=	num_char7[3][3];
output_data_map[558]	=	num_char7[3][4];
output_data_map[557]	=	num_char7[3][5];
output_data_map[556]	=	num_char7[3][6];
output_data_map[555]	=	num_char7[3][7];
output_data_map[584]	=	num_char7[4][0];
output_data_map[583]	=	num_char7[4][1];
output_data_map[582]	=	num_char7[4][2];
output_data_map[581]	=	num_char7[4][3];
output_data_map[580]	=	num_char7[4][4];
output_data_map[579]	=	num_char7[4][5];
output_data_map[578]	=	num_char7[4][6];
output_data_map[577]	=	num_char7[4][7];
break;		
case	8:	
output_data_map[464]	=	num_char8[0][0];
output_data_map[463]	=	num_char8[0][1];
output_data_map[462]	=	num_char8[0][2];
output_data_map[461]	=	num_char8[0][3];
output_data_map[460]	=	num_char8[0][4];
output_data_map[459]	=	num_char8[0][5];
output_data_map[458]	=	num_char8[0][6];
output_data_map[457]	=	num_char8[0][7];
output_data_map[502]	=	num_char8[1][0];
output_data_map[501]	=	num_char8[1][1];
output_data_map[500]	=	num_char8[1][2];
output_data_map[499]	=	num_char8[1][3];
output_data_map[498]	=	num_char8[1][4];
output_data_map[497]	=	num_char8[1][5];
output_data_map[496]	=	num_char8[1][6];
output_data_map[495]	=	num_char8[1][7];
output_data_map[524]	=	num_char8[2][0];
output_data_map[523]	=	num_char8[2][1];
output_data_map[522]	=	num_char8[2][2];
output_data_map[521]	=	num_char8[2][3];
output_data_map[520]	=	num_char8[2][4];
output_data_map[519]	=	num_char8[2][5];
output_data_map[518]	=	num_char8[2][6];
output_data_map[517]	=	num_char8[2][7];
output_data_map[562]	=	num_char8[3][0];
output_data_map[561]	=	num_char8[3][1];
output_data_map[560]	=	num_char8[3][2];
output_data_map[559]	=	num_char8[3][3];
output_data_map[558]	=	num_char8[3][4];
output_data_map[557]	=	num_char8[3][5];
output_data_map[556]	=	num_char8[3][6];
output_data_map[555]	=	num_char8[3][7];
output_data_map[584]	=	num_char8[4][0];
output_data_map[583]	=	num_char8[4][1];
output_data_map[582]	=	num_char8[4][2];
output_data_map[581]	=	num_char8[4][3];
output_data_map[580]	=	num_char8[4][4];
output_data_map[579]	=	num_char8[4][5];
output_data_map[578]	=	num_char8[4][6];
output_data_map[577]	=	num_char8[4][7];
break;		
case	9:	
output_data_map[464]	=	num_char9[0][0];
output_data_map[463]	=	num_char9[0][1];
output_data_map[462]	=	num_char9[0][2];
output_data_map[461]	=	num_char9[0][3];
output_data_map[460]	=	num_char9[0][4];
output_data_map[459]	=	num_char9[0][5];
output_data_map[458]	=	num_char9[0][6];
output_data_map[457]	=	num_char9[0][7];
output_data_map[502]	=	num_char9[1][0];
output_data_map[501]	=	num_char9[1][1];
output_data_map[500]	=	num_char9[1][2];
output_data_map[499]	=	num_char9[1][3];
output_data_map[498]	=	num_char9[1][4];
output_data_map[497]	=	num_char9[1][5];
output_data_map[496]	=	num_char9[1][6];
output_data_map[495]	=	num_char9[1][7];
output_data_map[524]	=	num_char9[2][0];
output_data_map[523]	=	num_char9[2][1];
output_data_map[522]	=	num_char9[2][2];
output_data_map[521]	=	num_char9[2][3];
output_data_map[520]	=	num_char9[2][4];
output_data_map[519]	=	num_char9[2][5];
output_data_map[518]	=	num_char9[2][6];
output_data_map[517]	=	num_char9[2][7];
output_data_map[562]	=	num_char9[3][0];
output_data_map[561]	=	num_char9[3][1];
output_data_map[560]	=	num_char9[3][2];
output_data_map[559]	=	num_char9[3][3];
output_data_map[558]	=	num_char9[3][4];
output_data_map[557]	=	num_char9[3][5];
output_data_map[556]	=	num_char9[3][6];
output_data_map[555]	=	num_char9[3][7];
output_data_map[584]	=	num_char9[4][0];
output_data_map[583]	=	num_char9[4][1];
output_data_map[582]	=	num_char9[4][2];
output_data_map[581]	=	num_char9[4][3];
output_data_map[580]	=	num_char9[4][4];
output_data_map[579]	=	num_char9[4][5];
output_data_map[578]	=	num_char9[4][6];
output_data_map[577]	=	num_char9[4][7];
break;		
}		
  switch(right_char){
  case 0:
    output_data_map[675] =  num_char0[0][0];
    output_data_map[676] =  num_char0[0][1];
    output_data_map[677] =  num_char0[0][2];
    output_data_map[678] =  num_char0[0][3];
    output_data_map[679] =  num_char0[0][4];
    output_data_map[680] =  num_char0[0][5];
    output_data_map[681] =  num_char0[0][6];
    output_data_map[682] =  num_char0[0][7];
    output_data_map[697] =  num_char0[1][0];
    output_data_map[698] =  num_char0[1][1];
    output_data_map[699] =  num_char0[1][2];
    output_data_map[700] =  num_char0[1][3];
    output_data_map[701] =  num_char0[1][4];
    output_data_map[702] =  num_char0[1][5];
    output_data_map[703] =  num_char0[1][6];
    output_data_map[704] =  num_char0[1][7];
    output_data_map[735] =  num_char0[2][0];
    output_data_map[736] =  num_char0[2][1];
    output_data_map[737] =  num_char0[2][2];
    output_data_map[738] =  num_char0[2][3];
    output_data_map[739] =  num_char0[2][4];
    output_data_map[740] =  num_char0[2][5];
    output_data_map[741] =  num_char0[2][6];
    output_data_map[742] =  num_char0[2][7];
    output_data_map[757] =  num_char0[3][0];
    output_data_map[758] =  num_char0[3][1];
    output_data_map[759] =  num_char0[3][2];
    output_data_map[760] =  num_char0[3][3];
    output_data_map[761] =  num_char0[3][4];
    output_data_map[762] =  num_char0[3][5];
    output_data_map[763] =  num_char0[3][6];
    output_data_map[764] =  num_char0[3][7];
    output_data_map[795] =  num_char0[4][0];
    output_data_map[796] =  num_char0[4][1];
    output_data_map[797] =  num_char0[4][2];
    output_data_map[798] =  num_char0[4][3];
    output_data_map[799] =  num_char0[4][4];
    output_data_map[800] =  num_char0[4][5];
    output_data_map[801] =  num_char0[4][6];
    output_data_map[802] =  num_char0[4][7];
    break;
  case 1:
    output_data_map[675] =  num_char1[0][0];
    output_data_map[676] =  num_char1[0][1];
    output_data_map[677] =  num_char1[0][2];
    output_data_map[678] =  num_char1[0][3];
    output_data_map[679] =  num_char1[0][4];
    output_data_map[680] =  num_char1[0][5];
    output_data_map[681] =  num_char1[0][6];
    output_data_map[682] =  num_char1[0][7];
    output_data_map[697] =  num_char1[1][0];
    output_data_map[698] =  num_char1[1][1];
    output_data_map[699] =  num_char1[1][2];
    output_data_map[700] =  num_char1[1][3];
    output_data_map[701] =  num_char1[1][4];
    output_data_map[702] =  num_char1[1][5];
    output_data_map[703] =  num_char1[1][6];
    output_data_map[704] =  num_char1[1][7];
    output_data_map[735] =  num_char1[2][0];
    output_data_map[736] =  num_char1[2][1];
    output_data_map[737] =  num_char1[2][2];
    output_data_map[738] =  num_char1[2][3];
    output_data_map[739] =  num_char1[2][4];
    output_data_map[740] =  num_char1[2][5];
    output_data_map[741] =  num_char1[2][6];
    output_data_map[742] =  num_char1[2][7];
    output_data_map[757] =  num_char1[3][0];
    output_data_map[758] =  num_char1[3][1];
    output_data_map[759] =  num_char1[3][2];
    output_data_map[760] =  num_char1[3][3];
    output_data_map[761] =  num_char1[3][4];
    output_data_map[762] =  num_char1[3][5];
    output_data_map[763] =  num_char1[3][6];
    output_data_map[764] =  num_char1[3][7];
    output_data_map[795] =  num_char1[4][0];
    output_data_map[796] =  num_char1[4][1];
    output_data_map[797] =  num_char1[4][2];
    output_data_map[798] =  num_char1[4][3];
    output_data_map[799] =  num_char1[4][4];
    output_data_map[800] =  num_char1[4][5];
    output_data_map[801] =  num_char1[4][6];
    output_data_map[802] =  num_char1[4][7];
    break;
  case 2:
    output_data_map[675] =  num_char2[0][0];
    output_data_map[676] =  num_char2[0][1];
    output_data_map[677] =  num_char2[0][2];
    output_data_map[678] =  num_char2[0][3];
    output_data_map[679] =  num_char2[0][4];
    output_data_map[680] =  num_char2[0][5];
    output_data_map[681] =  num_char2[0][6];
    output_data_map[682] =  num_char2[0][7];
    output_data_map[697] =  num_char2[1][0];
    output_data_map[698] =  num_char2[1][1];
    output_data_map[699] =  num_char2[1][2];
    output_data_map[700] =  num_char2[1][3];
    output_data_map[701] =  num_char2[1][4];
    output_data_map[702] =  num_char2[1][5];
    output_data_map[703] =  num_char2[1][6];
    output_data_map[704] =  num_char2[1][7];
    output_data_map[735] =  num_char2[2][0];
    output_data_map[736] =  num_char2[2][1];
    output_data_map[737] =  num_char2[2][2];
    output_data_map[738] =  num_char2[2][3];
    output_data_map[739] =  num_char2[2][4];
    output_data_map[740] =  num_char2[2][5];
    output_data_map[741] =  num_char2[2][6];
    output_data_map[742] =  num_char2[2][7];
    output_data_map[757] =  num_char2[3][0];
    output_data_map[758] =  num_char2[3][1];
    output_data_map[759] =  num_char2[3][2];
    output_data_map[760] =  num_char2[3][3];
    output_data_map[761] =  num_char2[3][4];
    output_data_map[762] =  num_char2[3][5];
    output_data_map[763] =  num_char2[3][6];
    output_data_map[764] =  num_char2[3][7];
    output_data_map[795] =  num_char2[4][0];
    output_data_map[796] =  num_char2[4][1];
    output_data_map[797] =  num_char2[4][2];
    output_data_map[798] =  num_char2[4][3];
    output_data_map[799] =  num_char2[4][4];
    output_data_map[800] =  num_char2[4][5];
    output_data_map[801] =  num_char2[4][6];
    output_data_map[802] =  num_char2[4][7];
    break;
  case 3:
    output_data_map[675] =  num_char3[0][0];
    output_data_map[676] =  num_char3[0][1];
    output_data_map[677] =  num_char3[0][2];
    output_data_map[678] =  num_char3[0][3];
    output_data_map[679] =  num_char3[0][4];
    output_data_map[680] =  num_char3[0][5];
    output_data_map[681] =  num_char3[0][6];
    output_data_map[682] =  num_char3[0][7];
    output_data_map[697] =  num_char3[1][0];
    output_data_map[698] =  num_char3[1][1];
    output_data_map[699] =  num_char3[1][2];
    output_data_map[700] =  num_char3[1][3];
    output_data_map[701] =  num_char3[1][4];
    output_data_map[702] =  num_char3[1][5];
    output_data_map[703] =  num_char3[1][6];
    output_data_map[704] =  num_char3[1][7];
    output_data_map[735] =  num_char3[2][0];
    output_data_map[736] =  num_char3[2][1];
    output_data_map[737] =  num_char3[2][2];
    output_data_map[738] =  num_char3[2][3];
    output_data_map[739] =  num_char3[2][4];
    output_data_map[740] =  num_char3[2][5];
    output_data_map[741] =  num_char3[2][6];
    output_data_map[742] =  num_char3[2][7];
    output_data_map[757] =  num_char3[3][0];
    output_data_map[758] =  num_char3[3][1];
    output_data_map[759] =  num_char3[3][2];
    output_data_map[760] =  num_char3[3][3];
    output_data_map[761] =  num_char3[3][4];
    output_data_map[762] =  num_char3[3][5];
    output_data_map[763] =  num_char3[3][6];
    output_data_map[764] =  num_char3[3][7];
    output_data_map[795] =  num_char3[4][0];
    output_data_map[796] =  num_char3[4][1];
    output_data_map[797] =  num_char3[4][2];
    output_data_map[798] =  num_char3[4][3];
    output_data_map[799] =  num_char3[4][4];
    output_data_map[800] =  num_char3[4][5];
    output_data_map[801] =  num_char3[4][6];
    output_data_map[802] =  num_char3[4][7];
    break;
  case 4:
    output_data_map[675] =  num_char4[0][0];
    output_data_map[676] =  num_char4[0][1];
    output_data_map[677] =  num_char4[0][2];
    output_data_map[678] =  num_char4[0][3];
    output_data_map[679] =  num_char4[0][4];
    output_data_map[680] =  num_char4[0][5];
    output_data_map[681] =  num_char4[0][6];
    output_data_map[682] =  num_char4[0][7];
    output_data_map[697] =  num_char4[1][0];
    output_data_map[698] =  num_char4[1][1];
    output_data_map[699] =  num_char4[1][2];
    output_data_map[700] =  num_char4[1][3];
    output_data_map[701] =  num_char4[1][4];
    output_data_map[702] =  num_char4[1][5];
    output_data_map[703] =  num_char4[1][6];
    output_data_map[704] =  num_char4[1][7];
    output_data_map[735] =  num_char4[2][0];
    output_data_map[736] =  num_char4[2][1];
    output_data_map[737] =  num_char4[2][2];
    output_data_map[738] =  num_char4[2][3];
    output_data_map[739] =  num_char4[2][4];
    output_data_map[740] =  num_char4[2][5];
    output_data_map[741] =  num_char4[2][6];
    output_data_map[742] =  num_char4[2][7];
    output_data_map[757] =  num_char4[3][0];
    output_data_map[758] =  num_char4[3][1];
    output_data_map[759] =  num_char4[3][2];
    output_data_map[760] =  num_char4[3][3];
    output_data_map[761] =  num_char4[3][4];
    output_data_map[762] =  num_char4[3][5];
    output_data_map[763] =  num_char4[3][6];
    output_data_map[764] =  num_char4[3][7];
    output_data_map[795] =  num_char4[4][0];
    output_data_map[796] =  num_char4[4][1];
    output_data_map[797] =  num_char4[4][2];
    output_data_map[798] =  num_char4[4][3];
    output_data_map[799] =  num_char4[4][4];
    output_data_map[800] =  num_char4[4][5];
    output_data_map[801] =  num_char4[4][6];
    output_data_map[802] =  num_char4[4][7];
    break;
  case 5:
    output_data_map[675] =  num_char5[0][0];
    output_data_map[676] =  num_char5[0][1];
    output_data_map[677] =  num_char5[0][2];
    output_data_map[678] =  num_char5[0][3];
    output_data_map[679] =  num_char5[0][4];
    output_data_map[680] =  num_char5[0][5];
    output_data_map[681] =  num_char5[0][6];
    output_data_map[682] =  num_char5[0][7];
    output_data_map[697] =  num_char5[1][0];
    output_data_map[698] =  num_char5[1][1];
    output_data_map[699] =  num_char5[1][2];
    output_data_map[700] =  num_char5[1][3];
    output_data_map[701] =  num_char5[1][4];
    output_data_map[702] =  num_char5[1][5];
    output_data_map[703] =  num_char5[1][6];
    output_data_map[704] =  num_char5[1][7];
    output_data_map[735] =  num_char5[2][0];
    output_data_map[736] =  num_char5[2][1];
    output_data_map[737] =  num_char5[2][2];
    output_data_map[738] =  num_char5[2][3];
    output_data_map[739] =  num_char5[2][4];
    output_data_map[740] =  num_char5[2][5];
    output_data_map[741] =  num_char5[2][6];
    output_data_map[742] =  num_char5[2][7];
    output_data_map[757] =  num_char5[3][0];
    output_data_map[758] =  num_char5[3][1];
    output_data_map[759] =  num_char5[3][2];
    output_data_map[760] =  num_char5[3][3];
    output_data_map[761] =  num_char5[3][4];
    output_data_map[762] =  num_char5[3][5];
    output_data_map[763] =  num_char5[3][6];
    output_data_map[764] =  num_char5[3][7];
    output_data_map[795] =  num_char5[4][0];
    output_data_map[796] =  num_char5[4][1];
    output_data_map[797] =  num_char5[4][2];
    output_data_map[798] =  num_char5[4][3];
    output_data_map[799] =  num_char5[4][4];
    output_data_map[800] =  num_char5[4][5];
    output_data_map[801] =  num_char5[4][6];
    output_data_map[802] =  num_char5[4][7];
    break;
  case 6:
    output_data_map[675] =  num_char6[0][0];
    output_data_map[676] =  num_char6[0][1];
    output_data_map[677] =  num_char6[0][2];
    output_data_map[678] =  num_char6[0][3];
    output_data_map[679] =  num_char6[0][4];
    output_data_map[680] =  num_char6[0][5];
    output_data_map[681] =  num_char6[0][6];
    output_data_map[682] =  num_char6[0][7];
    output_data_map[697] =  num_char6[1][0];
    output_data_map[698] =  num_char6[1][1];
    output_data_map[699] =  num_char6[1][2];
    output_data_map[700] =  num_char6[1][3];
    output_data_map[701] =  num_char6[1][4];
    output_data_map[702] =  num_char6[1][5];
    output_data_map[703] =  num_char6[1][6];
    output_data_map[704] =  num_char6[1][7];
    output_data_map[735] =  num_char6[2][0];
    output_data_map[736] =  num_char6[2][1];
    output_data_map[737] =  num_char6[2][2];
    output_data_map[738] =  num_char6[2][3];
    output_data_map[739] =  num_char6[2][4];
    output_data_map[740] =  num_char6[2][5];
    output_data_map[741] =  num_char6[2][6];
    output_data_map[742] =  num_char6[2][7];
    output_data_map[757] =  num_char6[3][0];
    output_data_map[758] =  num_char6[3][1];
    output_data_map[759] =  num_char6[3][2];
    output_data_map[760] =  num_char6[3][3];
    output_data_map[761] =  num_char6[3][4];
    output_data_map[762] =  num_char6[3][5];
    output_data_map[763] =  num_char6[3][6];
    output_data_map[764] =  num_char6[3][7];
    output_data_map[795] =  num_char6[4][0];
    output_data_map[796] =  num_char6[4][1];
    output_data_map[797] =  num_char6[4][2];
    output_data_map[798] =  num_char6[4][3];
    output_data_map[799] =  num_char6[4][4];
    output_data_map[800] =  num_char6[4][5];
    output_data_map[801] =  num_char6[4][6];
    output_data_map[802] =  num_char6[4][7];
    break;
  case 7:
    output_data_map[675] =  num_char7[0][0];
    output_data_map[676] =  num_char7[0][1];
    output_data_map[677] =  num_char7[0][2];
    output_data_map[678] =  num_char7[0][3];
    output_data_map[679] =  num_char7[0][4];
    output_data_map[680] =  num_char7[0][5];
    output_data_map[681] =  num_char7[0][6];
    output_data_map[682] =  num_char7[0][7];
    output_data_map[697] =  num_char7[1][0];
    output_data_map[698] =  num_char7[1][1];
    output_data_map[699] =  num_char7[1][2];
    output_data_map[700] =  num_char7[1][3];
    output_data_map[701] =  num_char7[1][4];
    output_data_map[702] =  num_char7[1][5];
    output_data_map[703] =  num_char7[1][6];
    output_data_map[704] =  num_char7[1][7];
    output_data_map[735] =  num_char7[2][0];
    output_data_map[736] =  num_char7[2][1];
    output_data_map[737] =  num_char7[2][2];
    output_data_map[738] =  num_char7[2][3];
    output_data_map[739] =  num_char7[2][4];
    output_data_map[740] =  num_char7[2][5];
    output_data_map[741] =  num_char7[2][6];
    output_data_map[742] =  num_char7[2][7];
    output_data_map[757] =  num_char7[3][0];
    output_data_map[758] =  num_char7[3][1];
    output_data_map[759] =  num_char7[3][2];
    output_data_map[760] =  num_char7[3][3];
    output_data_map[761] =  num_char7[3][4];
    output_data_map[762] =  num_char7[3][5];
    output_data_map[763] =  num_char7[3][6];
    output_data_map[764] =  num_char7[3][7];
    output_data_map[795] =  num_char7[4][0];
    output_data_map[796] =  num_char7[4][1];
    output_data_map[797] =  num_char7[4][2];
    output_data_map[798] =  num_char7[4][3];
    output_data_map[799] =  num_char7[4][4];
    output_data_map[800] =  num_char7[4][5];
    output_data_map[801] =  num_char7[4][6];
    output_data_map[802] =  num_char7[4][7];
    break;
  case 8:
    output_data_map[675] =  num_char8[0][0];
    output_data_map[676] =  num_char8[0][1];
    output_data_map[677] =  num_char8[0][2];
    output_data_map[678] =  num_char8[0][3];
    output_data_map[679] =  num_char8[0][4];
    output_data_map[680] =  num_char8[0][5];
    output_data_map[681] =  num_char8[0][6];
    output_data_map[682] =  num_char8[0][7];
    output_data_map[697] =  num_char8[1][0];
    output_data_map[698] =  num_char8[1][1];
    output_data_map[699] =  num_char8[1][2];
    output_data_map[700] =  num_char8[1][3];
    output_data_map[701] =  num_char8[1][4];
    output_data_map[702] =  num_char8[1][5];
    output_data_map[703] =  num_char8[1][6];
    output_data_map[704] =  num_char8[1][7];
    output_data_map[735] =  num_char8[2][0];
    output_data_map[736] =  num_char8[2][1];
    output_data_map[737] =  num_char8[2][2];
    output_data_map[738] =  num_char8[2][3];
    output_data_map[739] =  num_char8[2][4];
    output_data_map[740] =  num_char8[2][5];
    output_data_map[741] =  num_char8[2][6];
    output_data_map[742] =  num_char8[2][7];
    output_data_map[757] =  num_char8[3][0];
    output_data_map[758] =  num_char8[3][1];
    output_data_map[759] =  num_char8[3][2];
    output_data_map[760] =  num_char8[3][3];
    output_data_map[761] =  num_char8[3][4];
    output_data_map[762] =  num_char8[3][5];
    output_data_map[763] =  num_char8[3][6];
    output_data_map[764] =  num_char8[3][7]; 
    output_data_map[795] =  num_char8[4][0];
    output_data_map[796] =  num_char8[4][1];
    output_data_map[797] =  num_char8[4][2];
    output_data_map[798] =  num_char8[4][3];
    output_data_map[799] =  num_char8[4][4];
    output_data_map[800] =  num_char8[4][5];
    output_data_map[801] =  num_char8[4][6];
    output_data_map[802] =  num_char8[4][7];
    break;
  case 9:
    output_data_map[675] =  num_char9[0][0];
    output_data_map[676] =  num_char9[0][1];
    output_data_map[677] =  num_char9[0][2];
    output_data_map[678] =  num_char9[0][3];
    output_data_map[679] =  num_char9[0][4];
    output_data_map[680] =  num_char9[0][5];
    output_data_map[681] =  num_char9[0][6];
    output_data_map[682] =  num_char9[0][7];
    output_data_map[697] =  num_char9[1][0];
    output_data_map[698] =  num_char9[1][1];
    output_data_map[699] =  num_char9[1][2];
    output_data_map[700] =  num_char9[1][3];
    output_data_map[701] =  num_char9[1][4];
    output_data_map[702] =  num_char9[1][5];
    output_data_map[703] =  num_char9[1][6];
    output_data_map[704] =  num_char9[1][7];
    output_data_map[735] =  num_char9[2][0];
    output_data_map[736] =  num_char9[2][1];
    output_data_map[737] =  num_char9[2][2];
    output_data_map[738] =  num_char9[2][3];
    output_data_map[739] =  num_char9[2][4];
    output_data_map[740] =  num_char9[2][5];
    output_data_map[741] =  num_char9[2][6];
    output_data_map[742] =  num_char9[2][7];
    output_data_map[757] =  num_char9[3][0];
    output_data_map[758] =  num_char9[3][1];
    output_data_map[759] =  num_char9[3][2];
    output_data_map[760] =  num_char9[3][3];
    output_data_map[761] =  num_char9[3][4];
    output_data_map[762] =  num_char9[3][5];
    output_data_map[763] =  num_char9[3][6];
    output_data_map[764] =  num_char9[3][7];
    output_data_map[795] =  num_char9[4][0];
    output_data_map[796] =  num_char9[4][1];
    output_data_map[797] =  num_char9[4][2];
    output_data_map[798] =  num_char9[4][3];
    output_data_map[799] =  num_char9[4][4];
    output_data_map[800] =  num_char9[4][5];
    output_data_map[801] =  num_char9[4][6];
    output_data_map[802] =  num_char9[4][7];
    break;  
  }
} 

//*************************************************************************************************// 
void Block_map_update(){ 
  int z = 0; 
  for ( int x = 60; x < 360; x++) 
  { 
    output_data_map[x] = Block_datamap[z]; 
    z++; 
  }
} 

//*************************************************************************************************//  
void set_default_master_data_map(){
  for (int x = 0; x < LED_total_count; x++)
  {
    output_data_map[x] = master_data_map[x];
  }
}

//*************************************************************************************************// 
void clear_master_data_map(){
  for (int x = 0; x < LED_total_count; x++)
  {
    output_data_map[x] = default_data_map[x];
  }
}

//*************************************************************************************************// 
void generate_next_piece_gfx()
{
  if (next_piece != 7)
  {
  next_piece = random(7);
  }
  switch (next_piece){
  case 0:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 1;
    output_data_map[484] = 1;
    output_data_map[483] = 1;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 0;
    output_data_map[534] = 1;
    output_data_map[535] = 1;
    output_data_map[536] = 1;
    output_data_map[537] = 0;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 0;
    output_data_map[545] = 1;
    output_data_map[544] = 1;
    output_data_map[543] = 1;
    output_data_map[542] = 0;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 0;
    output_data_map[594] = 1;
    output_data_map[595] = 1;
    output_data_map[596] = 1;
    output_data_map[597] = 0;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 0;
    output_data_map[605] = 1;
    output_data_map[604] = 1;
    output_data_map[603] = 1;
    output_data_map[602] = 0;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 0;
    output_data_map[654] = 1;
    output_data_map[655] = 1;
    output_data_map[656] = 1;
    output_data_map[657] = 0;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 0;
    output_data_map[665] = 1;
    output_data_map[664] = 1;
    output_data_map[663] = 1;
    output_data_map[662] = 0;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 0;
    output_data_map[714] = 1;
    output_data_map[715] = 1;
    output_data_map[716] = 1;
    output_data_map[717] = 0;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 0;
    output_data_map[725] = 1;
    output_data_map[724] = 1;
    output_data_map[723] = 1;
    output_data_map[722] = 0;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 0;
    output_data_map[774] = 1;
    output_data_map[775] = 1;
    output_data_map[776] = 1;
    output_data_map[777] = 0;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 1;
    output_data_map[784] = 1;
    output_data_map[783] = 1;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
  case 1:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 0;
    output_data_map[484] = 0;
    output_data_map[483] = 0;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 2;
    output_data_map[534] = 2;
    output_data_map[535] = 2;
    output_data_map[536] = 2;
    output_data_map[537] = 2;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 2;
    output_data_map[545] = 2;
    output_data_map[544] = 2;
    output_data_map[543] = 2;
    output_data_map[542] = 2;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 0;
    output_data_map[594] = 0;
    output_data_map[595] = 0;
    output_data_map[596] = 2;
    output_data_map[597] = 2;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 0;
    output_data_map[605] = 0;
    output_data_map[604] = 0;
    output_data_map[603] = 2;
    output_data_map[602] = 2;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 0;
    output_data_map[654] = 0;
    output_data_map[655] = 0;
    output_data_map[656] = 2;
    output_data_map[657] = 2;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 0;
    output_data_map[665] = 0;
    output_data_map[664] = 0;
    output_data_map[663] = 2;
    output_data_map[662] = 2;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 0;
    output_data_map[714] = 0;
    output_data_map[715] = 0;
    output_data_map[716] = 2;
    output_data_map[717] = 2;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 0;
    output_data_map[725] = 0;
    output_data_map[724] = 0;
    output_data_map[723] = 2;
    output_data_map[722] = 2;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 0;
    output_data_map[774] = 0;
    output_data_map[775] = 0;
    output_data_map[776] = 2;
    output_data_map[777] = 2;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 0;
    output_data_map[784] = 0;
    output_data_map[783] = 0;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
    case 2:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 0;
    output_data_map[484] = 0;
    output_data_map[483] = 0;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 0;
    output_data_map[534] = 0;
    output_data_map[535] = 0;
    output_data_map[536] = 3;
    output_data_map[537] = 3;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 0;
    output_data_map[545] = 0;
    output_data_map[544] = 0;
    output_data_map[543] = 3;
    output_data_map[542] = 3;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 0;
    output_data_map[594] = 0;
    output_data_map[595] = 0;
    output_data_map[596] = 3;
    output_data_map[597] = 3;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 0;
    output_data_map[605] = 0;
    output_data_map[604] = 0;
    output_data_map[603] = 3;
    output_data_map[602] = 3;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 0;
    output_data_map[654] = 0;
    output_data_map[655] = 0;
    output_data_map[656] = 3;
    output_data_map[657] = 3;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 0;
    output_data_map[665] = 0;
    output_data_map[664] = 0;
    output_data_map[663] = 3;
    output_data_map[662] = 3;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 0;
    output_data_map[714] = 0;
    output_data_map[715] = 0;
    output_data_map[716] = 3;
    output_data_map[717] = 3;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 3;
    output_data_map[725] = 3;
    output_data_map[724] = 3;
    output_data_map[723] = 3;
    output_data_map[722] = 3;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 3;
    output_data_map[774] = 3;
    output_data_map[775] = 3;
    output_data_map[776] = 3;
    output_data_map[777] = 3;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 0;
    output_data_map[784] = 0;
    output_data_map[783] = 0;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
  case 3:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 0;
    output_data_map[484] = 0;
    output_data_map[483] = 0;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 0;
    output_data_map[534] = 0;
    output_data_map[535] = 0;
    output_data_map[536] = 0;
    output_data_map[537] = 0;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 0;
    output_data_map[545] = 0;
    output_data_map[544] = 0;
    output_data_map[543] = 0;
    output_data_map[542] = 0;
    output_data_map[541] = 0;
    output_data_map[592] = 4;
    output_data_map[593] = 4;
    output_data_map[594] = 4;
    output_data_map[595] = 4;
    output_data_map[596] = 4;
    output_data_map[597] = 4;
    output_data_map[598] = 4;
    output_data_map[607] = 4;
    output_data_map[606] = 4;
    output_data_map[605] = 4;
    output_data_map[604] = 4;
    output_data_map[603] = 4;
    output_data_map[602] = 4;
    output_data_map[601] = 4;
    output_data_map[652] = 4;
    output_data_map[653] = 4;
    output_data_map[654] = 4;
    output_data_map[655] = 4;
    output_data_map[656] = 4;
    output_data_map[657] = 4;
    output_data_map[658] = 4;
    output_data_map[667] = 4;
    output_data_map[666] = 4;
    output_data_map[665] = 4;
    output_data_map[664] = 4;
    output_data_map[663] = 4;
    output_data_map[662] = 4;
    output_data_map[661] = 4;
    output_data_map[712] = 4;
    output_data_map[713] = 4;
    output_data_map[714] = 4;
    output_data_map[715] = 4;
    output_data_map[716] = 4;
    output_data_map[717] = 4;
    output_data_map[718] = 4;
    output_data_map[727] = 0;
    output_data_map[726] = 0;
    output_data_map[725] = 0;
    output_data_map[724] = 0;
    output_data_map[723] = 0;
    output_data_map[722] = 0;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 0;
    output_data_map[774] = 0;
    output_data_map[775] = 0;
    output_data_map[776] = 0;
    output_data_map[777] = 0;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 0;
    output_data_map[784] = 0;
    output_data_map[783] = 0;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
  case 4:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 0;
    output_data_map[484] = 0;
    output_data_map[483] = 5;
    output_data_map[482] = 5;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 0;
    output_data_map[534] = 0;
    output_data_map[535] = 0;
    output_data_map[536] = 5;
    output_data_map[537] = 5;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 0;
    output_data_map[545] = 0;
    output_data_map[544] = 0;
    output_data_map[543] = 5;
    output_data_map[542] = 5;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 0;
    output_data_map[594] = 0;
    output_data_map[595] = 0;
    output_data_map[596] = 5;
    output_data_map[597] = 5;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 5;
    output_data_map[605] = 5;
    output_data_map[604] = 5;
    output_data_map[603] = 5;
    output_data_map[602] = 5;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 5;
    output_data_map[654] = 5;
    output_data_map[655] = 5;
    output_data_map[656] = 5;
    output_data_map[657] = 5;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 5;
    output_data_map[665] = 5;
    output_data_map[664] = 5;
    output_data_map[663] = 5;
    output_data_map[662] = 5;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 5;
    output_data_map[714] = 5;
    output_data_map[715] = 0;
    output_data_map[716] = 0;
    output_data_map[717] = 0;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 5;
    output_data_map[725] = 5;
    output_data_map[724] = 0;
    output_data_map[723] = 0;
    output_data_map[722] = 0;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 5;
    output_data_map[774] = 5;
    output_data_map[775] = 0;
    output_data_map[776] = 0;
    output_data_map[777] = 0;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 5;
    output_data_map[785] = 5;
    output_data_map[784] = 0;
    output_data_map[783] = 0;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
  case 5:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 0;
    output_data_map[484] = 0;
    output_data_map[483] = 0;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 0;
    output_data_map[534] = 0;
    output_data_map[535] = 6;
    output_data_map[536] = 6;
    output_data_map[537] = 6;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 0;
    output_data_map[545] = 0;
    output_data_map[544] = 6;
    output_data_map[543] = 6;
    output_data_map[542] = 6;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 0;
    output_data_map[594] = 0;
    output_data_map[595] = 6;
    output_data_map[596] = 6;
    output_data_map[597] = 6;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 6;
    output_data_map[605] = 6;
    output_data_map[604] = 6;
    output_data_map[603] = 6;
    output_data_map[602] = 6;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 6;
    output_data_map[654] = 6;
    output_data_map[655] = 6;
    output_data_map[656] = 6;
    output_data_map[657] = 6;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 6;
    output_data_map[665] = 6;
    output_data_map[664] = 6;
    output_data_map[663] = 6;
    output_data_map[662] = 6;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 0;
    output_data_map[714] = 0;
    output_data_map[715] = 6;
    output_data_map[716] = 6;
    output_data_map[717] = 6;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 0;
    output_data_map[725] = 0;
    output_data_map[724] = 6;
    output_data_map[723] = 6;
    output_data_map[722] = 6;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 0;
    output_data_map[774] = 0;
    output_data_map[775] = 6;
    output_data_map[776] = 6;
    output_data_map[777] = 6;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 0;
    output_data_map[784] = 0;
    output_data_map[783] = 0;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
  case 6:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 7;
    output_data_map[485] = 7;
    output_data_map[484] = 0;
    output_data_map[483] = 0;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 7;
    output_data_map[534] = 7;
    output_data_map[535] = 0;
    output_data_map[536] = 0;
    output_data_map[537] = 0;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 7;
    output_data_map[545] = 7;
    output_data_map[544] = 0;
    output_data_map[543] = 0;
    output_data_map[542] = 0;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 7;
    output_data_map[594] = 7;
    output_data_map[595] = 0;
    output_data_map[596] = 0;
    output_data_map[597] = 0;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 7;
    output_data_map[605] = 7;
    output_data_map[604] = 7;
    output_data_map[603] = 7;
    output_data_map[602] = 7;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 7;
    output_data_map[654] = 7;
    output_data_map[655] = 7;
    output_data_map[656] = 7;
    output_data_map[657] = 7;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 7;
    output_data_map[665] = 7;
    output_data_map[664] = 7;
    output_data_map[663] = 7;
    output_data_map[662] = 7;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 0;
    output_data_map[714] = 0;
    output_data_map[715] = 0;
    output_data_map[716] = 7;
    output_data_map[717] = 7;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 0;
    output_data_map[725] = 0;
    output_data_map[724] = 0;
    output_data_map[723] = 7;
    output_data_map[722] = 7;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 0;
    output_data_map[774] = 0;
    output_data_map[775] = 0;
    output_data_map[776] = 7;
    output_data_map[777] = 7;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 0;
    output_data_map[784] = 0;
    output_data_map[783] = 7;
    output_data_map[782] = 7;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
  case 7:
    output_data_map[472] = 0;
    output_data_map[473] = 0;
    output_data_map[474] = 0;
    output_data_map[475] = 0;
    output_data_map[476] = 0;
    output_data_map[477] = 0;
    output_data_map[478] = 0;
    output_data_map[487] = 0;
    output_data_map[486] = 0;
    output_data_map[485] = 0;
    output_data_map[484] = 0;
    output_data_map[483] = 0;
    output_data_map[482] = 0;
    output_data_map[481] = 0;
    output_data_map[532] = 0;
    output_data_map[533] = 0;
    output_data_map[534] = 0;
    output_data_map[535] = 0;
    output_data_map[536] = 0;
    output_data_map[537] = 0;
    output_data_map[538] = 0;
    output_data_map[547] = 0;
    output_data_map[546] = 0;
    output_data_map[545] = 0;
    output_data_map[544] = 0;
    output_data_map[543] = 0;
    output_data_map[542] = 0;
    output_data_map[541] = 0;
    output_data_map[592] = 0;
    output_data_map[593] = 0;
    output_data_map[594] = 0;
    output_data_map[595] = 0;
    output_data_map[596] = 0;
    output_data_map[597] = 0;
    output_data_map[598] = 0;
    output_data_map[607] = 0;
    output_data_map[606] = 0;
    output_data_map[605] = 0;
    output_data_map[604] = 0;
    output_data_map[603] = 0;
    output_data_map[602] = 0;
    output_data_map[601] = 0;
    output_data_map[652] = 0;
    output_data_map[653] = 0;
    output_data_map[654] = 0;
    output_data_map[655] = 0;
    output_data_map[656] = 0;
    output_data_map[657] = 0;
    output_data_map[658] = 0;
    output_data_map[667] = 0;
    output_data_map[666] = 0;
    output_data_map[665] = 0;
    output_data_map[664] = 0;
    output_data_map[663] = 0;
    output_data_map[662] = 0;
    output_data_map[661] = 0;
    output_data_map[712] = 0;
    output_data_map[713] = 0;
    output_data_map[714] = 0;
    output_data_map[715] = 0;
    output_data_map[716] = 0;
    output_data_map[717] = 0;
    output_data_map[718] = 0;
    output_data_map[727] = 0;
    output_data_map[726] = 0;
    output_data_map[725] = 0;
    output_data_map[724] = 0;
    output_data_map[723] = 0;
    output_data_map[722] = 0;
    output_data_map[721] = 0;
    output_data_map[772] = 0;
    output_data_map[773] = 0;
    output_data_map[774] = 0;
    output_data_map[775] = 0;
    output_data_map[776] = 0;
    output_data_map[777] = 0;
    output_data_map[778] = 0;
    output_data_map[787] = 0;
    output_data_map[786] = 0;
    output_data_map[785] = 0;
    output_data_map[784] = 0;
    output_data_map[783] = 0;
    output_data_map[782] = 0;
    output_data_map[781] = 0;
    output_data_map[832] = 0;
    output_data_map[833] = 0;
    output_data_map[834] = 0;
    output_data_map[835] = 0;
    output_data_map[836] = 0;
    output_data_map[837] = 0;
    output_data_map[838] = 0;
    break;
}
}
