//Eduardo Rivera ero6998
// Zombie Survival, defend your post as long as you can by // shooting the zombies.
// How to play
// Button 0 is to go up
// Button 1 is to do down
// Button 2 is to shoot, however only when moving up or down
// Button 3 is to buy powerup once your score is 255
// When hearts on top right reach zero, game is over. Recomplie // Page to restart
// IMPORTANT: To run smoothly, turn off the debug check //'Function clobbered callee saved register" 
// since I used some higher register as global variables.  //Otherwise the code 
// will stop frequently.


// ENJOY :)


// 320x240, 1024 bytes/row, 2 bytes per pixel: DE1-SoC
.equ WIDTH, 320
.equ HEIGHT, 240
.equ BUFFER_SIZE, 1024 * 240
.equ LOG2_BYTES_PER_ROW, 10
.equ LOG2_BYTES_PER_PIXEL, 1

.EQU PIX_BUFFER, 0xc8000000
.EQU TEXT_BUFFER, 0xc9000000
.EQU MAX_PIX_X, 0x0000013F
.EQU BUFFER_CONTROL, 0xff203020

// These are some useful defines that will help you access structure fields
.EQU PIXMAP_X, 0
.EQU PIXMAP_Y, 2
.EQU PIXMAP_WIDTH, 4
.EQU PIXMAP_HEIGHT, 6
.EQU PIXMAP_TRANSPARENCY, 8
.EQU PIXMAP_PIXELDATA, 10
.EQU PIXMAP_DIRECTION, 15





.global _start
_start:
	mov r0, #0 //This just clears all the registers manually so I don't have to reload to the page to 
	// run new program changes
	mov r1, #0
	mov r2, #0
	mov r3, #0
	mov r4, #0
	mov r5, #0
	mov r6, #0
	mov r7, #0
	mov r8, #0
	mov r9, #0
	mov r10, #0
	mov r11, #0
	
	//Here is where we set up the double buffer control
	ldr r0, =PIX_BUFFER
	ldr r1, =back_buff
	// Store first buffer address into buffer register
	ldr r2, =0xff203020
	str r0, [r2]
	// Load Back_Buffer address into backbuffer_register
	str r1, [r2, #4]
	
	ldr r10, =0xff203020 //r10 will store the address which we need to change in order
	// to swap bewteen the back buffer and pixel buffer
	mov r11, #1
	ldr r9, =back_buff
	
	//Print Main Messages to Main Menu
	ldr r3, =PIX_BUFFER
	bl ManageBuffers
	bl BlankScreen
	bl ClearTextBuffer
	mov r0, #30
	mov r1, #24
	ldr r2, =Main_Menu_Message
	bl DrawStr
	mov r0, #25
	mov r1, #40
	ldr r2, =Main_Menu_Buttons
	bl DrawStr
	mov r0, #0
	mov r1, #70
	ldr r2, =How_To_Play
	bl DrawStr
	str r1, [r10]
Main_Menu:

	
	ldr r4, =0xff200050 //load IO address into r4
	ldrb r5, [r4] 
	and r6, r5, #8
	cmp r6, #8
	beq Game_Setup


	b Main_Menu
	
Game_Setup:	
	//Clear everything again before moving into loop
	mov r0, #0 
	mov r1, #0
	mov r2, #0
	mov r4, #0
	mov r5, #0
	mov r6, #0
	mov r7, #0
	mov r8, #0
	ldr sp, =0x800000	// Initial stack pointer
	
	// R12 will act as a sort of timer to prevent the execution of the zombies moving
	// too quickly or the player as well.
	mov r12, #0
	
	//For bullet number released when shot
	ldr r8, =Bullets //Global Variable
	
	//Addres for hearts address, used to keep track of hearts
	ldr r7, =Hearts // Global Variable
	
	mov r6, #0 //Score global variable
	
	//Starts us off for the VGA control
	ldr r0, =SimplePix
	//ldr r3, =PIX_BUFFER
	bl ManageBuffers
	bl BlankScreen
	bl Draw_Zombies
	bl BitBlit
	bl ClearTextBuffer
	
	

	//Now we enter the main game loop which will run until 
	// You run out of health
Main_Game_Loop:
	// Change to correct buffer of the two
	bl ManageBuffers
	
	//Check User Input
	bl Check_Input
	
	

	// Move Sprites
	bl BlankScreen
	bl MoveSprite
	bl Bullets_Shot
	bl Horde_Approaches
	
	// Check for Collisions
	bl Check_Collisions
		
	// Update Score
	bl Update_Score
	
	// Draw all Sprites
	bl BitBlit
	bl Draw_Zombies
	bl Draw_Bullets
	bl Draw_Hearts
	
	
	//Now switch buffer
	add r12, r12, #1
	str r12, [r10]
    b Main_Game_Loop
	
	
Game_Over:
//Once you lose the game you are stuck here, you have reload 
// The page and start again. Running this game leads to
// some weird VGA action so reload page and rerun.
	ldr r10, =BUFFER_CONTROL
	//str r1, [r10]
	bl ManageBuffers
	bl BlankScreen
	bl ClearTextBuffer
	mov r0, #30
	mov r1, #50
	ldr r2, =Game_Over_Message
	bl DrawStr
	str r1, [r10]
game_over:
	b game_over
	
	
	
	
	
Update_Score:
	push {r0, r1, r2, r4, r5, r6, r7, lr}
	mov r0, #30
	mov r1, #2
	ldr r4, =Score
	ldrh r2, [r4]
	cmp r2, #255
	ble continue_iter_score
	mov r2, #255
	strh r2, [r4]
continue_iter_score:
	bl DrawNum
	mov r0, #23
	mov r1, #2
	ldr r2, =ScoreStr
	bl DrawStr
	pop {r0, r1, r2, r4, r5, r6, r7, pc}

BlankScreen:
	// Blanks the screen
	push {r0, r1, r2, r4, lr}
	mov r4, r3
    ldr r2, =0x0000
	mov r0, #0
BSLoop:
	mov r1, #0
RowLoop:
    str r2, [r4, r1]
	add r1, r1, #4
    cmp r1, #640
    blo RowLoop
	add r4, r4, #1024
	add r0, r0, #1
	cmp r0, #240
	blo BSLoop
	pop {r0, r1, r2, r4, pc}

Wait_For_Drawing:
	push {r4, r5, r6, r7, r8, r9, r10, r11, lr}
check_done_drawing:
	ldr r4, =BUFFER_CONTROL // r4 holds address 0xff203020
	ldr r5, [r4, #12] //Accesses the status register
	and r6, r5, #1 // Isolates the S status bit so we can check if the drawing is done
	cmp r6, #0
	beq drawing_done
	b check_done_drawing
	//We only enter done_drawing if the S bit is equal to 0
drawing_done:
	pop {r4, r5, r6, r7, r8, r9, r10, r11, pc}

ManageBuffers:
	push {r4, r5, r6, r7, r8, r9, r10, r11, lr}
	bl Wait_For_Drawing
	ldr r5, =back_buff
	ldr r4, =BUFFER_CONTROL
	ldr r7, [r4]
	cmp r7, r5
	beq switch_back_to_pix
	ldr r3, =back_buff//if its not back_buff, then it has to be pix_buffer 
	//meaning we just switch back to back_buff
	b exit_func
switch_back_to_pix:
	ldr r3, =PIX_BUFFER
	// We need r3 to be either of the buffers so we can change it
	// so we store PIX_BUFFER into the back register so it doesn't draw
	// it while we change it, same with back_buff
exit_func:
	pop {r4, r5, r6, r7, r8, r9, r10, r11, pc}
	

Horde_Approaches:
	// This functions changes the coordinates for all 100 zombies
	// Takes no inputs and returns no outputs
	// It just simply changes the x and y coordinates of all 100 zombies
	// They all get closer to the player
	push {r4, r5, r6, r8, r9, r10, r11, r12, lr}
	ldr r4, =Zombies
	ldr r5, =3500
	add r5, r5, r4
	ldr r6, =SimplePix
	ldrh r8, [r6, #PIXMAP_X] // x coord of player
	b zombies_correct_track
move_zombies_loop:
	ldrh r10, [r4, #PIXMAP_X] // x coord of zombie
	ldr r11, =300
	cmp r10, r11 //Check to see if zombies have reached the player
	blt continue_moving_zombies
	mov r10, #0
	// Speed up zombie if they get to player to make the game harder
	ldr r11, =Velocity_of_Zombies
	ldrh r12, [r11]
	add r12, r12, #2
	strh r12, [r11]
	bl Lose_Heart
continue_moving_zombies:
	ldr r11, =Velocity_of_Zombies
	ldrh r12, [r11]
	add r10, r10, r12
	strh r10, [r4, #PIXMAP_X] //store new x into class
	add r4, r4, #28
zombies_correct_track:
	cmp r4, r5
	blt move_zombies_loop
	pop {r4, r5, r6, r8, r9, r10,r11, r12, pc}
	
Bullets_Shot:
	// This functions moves the bullets towards the left, and loops them 
	// back into infinite loop offscreen when they reach the end of the left screen
	push {r4, r5, r6, r7, r8, r9, r10, r11, lr}
	ldr r4, =Bullets
	ldr r5, =700
	add r5, r5, r4
	ldr r6, =321
	b bullets_correct_track
move_bullets_loop:
	ldrh r8, [r4, #PIXMAP_X] //x coord of bullets
	cmp r8, r6
	beq no_store_inf_loop
	add r8, r8, #-20
	cmp r8, #10
	bgt no_store_inf_loop
	mov r8, r6
no_store_inf_loop:
	strh r8, [r4, #PIXMAP_X]
	add r4, r4, #14
bullets_correct_track:
	cmp r4, r5
	blt move_bullets_loop
	pop {r4, r5, r6, r7, r8, r9, r10, r11, pc}
	
	

Draw_Bullets:
	//This function draws all the bullets
	// to make things less complicated the buttons will
	// be drawn in an infinite loop to the right offscreen
	// until they are shot from the player which they go back 
	// to the infinite loop again. MAX 25 bullets on screen at once
	push {r0, r4, r5, r6, r7, r8, r9, r10, lr}
	ldr r4, =Bullets
	ldr r5, =700
	add r5, r5, r4
	b bullets_done
draw_bullets_loop:
	mov r0, r4
	bl BitBlit
	add r4, r4, #14
bullets_done:
	cmp r4, r5
	ble draw_bullets_loop
	pop {r0, r4, r5, r6, r7, r8, r9, r10, pc}
	
Draw_Zombies:
	// This function draws all 100 zombies
	// Takes no inputs and returns no outputs
	// It just simply draws the zombies
	push {r0, r4, r5, r6, r7, r8, lr}
	ldr r4, =Zombies
	ldr r5, =3500
	add r5, r5, r4
	b zombies_done
draw_zombies_loop:
	mov r0, r4
	bl BitBlit
	add r4, r4, #28
zombies_done:
	cmp r4, r5
	ble draw_zombies_loop
	pop {r0, r4, r5, r6, r7, r8, pc}

Draw_Hearts:
	//Function draws the hearts in the top right corner
	push {r0, r4, r5, r6, r7, r8, r9, r10, r11, lr}
	ldr r4, =Hearts
	ldr r5, =350
	add r5, r5, r4
	b hearts_done
draw_hearts_loop:
	mov r0, r4
	bl BitBlit
	add r4, r4, #70
hearts_done:
	cmp r4, r5
	ble draw_hearts_loop
	pop {r0, r4, r5, r6, r7, r8, r9, r10, r11, pc}
	
Lose_Heart:
	push {r4, r5, r6, r8, r9, r10, lr}
	ldr r5, =350
	ldr r4, =Hearts
	add r5, r5, r4 //Max Address of Hearts
	cmp r7, r5
	blt still_have_hearts //if we reach the last heart, we lose the game
	//Game Over
	ldr r8, =321
	strh r8, [r7, #PIXMAP_X] 
	bl Game_Over
still_have_hearts:
	ldr r8, =321
	strh r8, [r7, #PIXMAP_X] //move hearts offscreen
	add r7, r7, #70
Continue_Game:
	pop {r4, r5, r6, r8, r9, r10, pc}
	

Check_Collisions:
	push {r2, r4, r5, r6, r7, r8, r9, r10, r11, r12, lr}
	ldr r4, =Zombies
	ldr r5, =3500
	add r5, r5, r4
	ldr r7, =Bullets
	ldr r2, =700
	add r2, r2, r7
	b zombie_col_check
check_zombie_loop:
	ldrh r8, [r4, #PIXMAP_X] //x coord of zombie
	ldrh r9, [r4, #PIXMAP_Y] // y coord of zombie
	b bullets_coll_check
coll_bullets_loop:
	ldrh r10, [r7, #PIXMAP_X] // x coord of bullet
	ldrh r11, [r7, #PIXMAP_Y] //y coord of bullet
	//first we check if the bullets are in the shooting range
	cmp r10, #10
	blt ignore_col
	ldr r12, =300
	cmp r10, r12
	ble bullet_in_range
	b ignore_col
bullet_in_range:
	sub r9, r9, #2
	cmp r9, r11 // if top of zombie below bullet, ignore
	bgt ignore_col
	add r9, r9, #10 //bottom left side of zombie
	cmp r9, r11 //if bottom of zombie is above bullet, ignore
	blt ignore_col
	add r8, r8, #4
	cmp r8, r10 //if right of zombie is less than bullet, ignore
	blt ignore_col
	//If we enter here then we for certain had a collision
	mov r10, #0
	strh r10, [r4, #PIXMAP_X]
	ldr r10, =321 
	strh r10, [r7, #PIXMAP_X]
	ldr r6, =Score
	ldrh r10, [r6]
	add r10, r10, #1
	strh r10, [r6]
ignore_col:
	ldrh r8, [r4, #PIXMAP_X] //x coord of zombie
	ldrh r9, [r4, #PIXMAP_Y] // y coord of zombie
	add r7, r7, #14
bullets_coll_check:
	cmp r7, r2
	ble coll_bullets_loop
	ldr r7, =Bullets
	add r4, r4, #28
zombie_col_check:
	cmp r4, r5
	ble check_zombie_loop
	pop {r2, r4, r5, r6, r7, r8, r9, r10, r11, r12, pc}


	
Check_Input:
	// This function will read in the current button being pressed and then return the 
	// direction via the sign and magnitude of r1 and r2
	// No parameters but returns x_velocity and y_velocity
	push {r4, r5, r6, r7, r9, r10, r11, r12, lr}
	ldr r4, =0xff200050 //load IO address into r4
	ldrb r5, [r4]  //Since the buttons are controlled by 4 bit, we need to load a btye max
	//Check for "up" arrow
	// Here we set the highest bullet we can shoot
	// r8 globally holds the address for bullet 
	ldr r6, =Bullets
	ldr r7, =350
	add r7, r7, r6
	
	//Check if powerup bought
	and r6, r5, #8
	cmp r6, #8
	bne power_not_bought
	ldr r10, =Score
	ldrh r11, [r10]
	cmp r11, #255
	blt power_not_bought
	ldr r10, =Velocity_of_Zombies
	ldrh r11, [r10]
	mov r11, #1
	strh r11, [r10]
	ldr r10, =Score
	mov r11, #0
	strh r11, [r10]
power_not_bought:
	// BY default, no button push means there is no mov in r1 or r2
	mov r1, #0
	mov r2, #0
	//Now we check and change values of x and y if we pressed the push buttons
	and r6, r5, #1
	cmp r6, #1
	bne no_down_movement
	mov r2, #-10
no_down_movement:
	and r6, r5, #2
	cmp r6, #2
	bne no_up_movement
	mov r2, #10
no_up_movement:
//Now we need to check if the player goes off the screen which we will make them
// kind of bounce off the screen
	ldr r6, =225
	ldrh r9, [r0, #PIXMAP_Y]
	cmp r9, r6
	ble continue_as_normal
	ldr r6, =220
	strh r6, [r0, #PIXMAP_Y]
continue_as_normal:
	cmp r9, #20
	bgt continue_as_nor
	mov r6, #25
	strh r6, [r0, #PIXMAP_Y]
continue_as_nor:
//Check for bullet shooting, which is stored in r8 globally
	and r6, r5, #4
	cmp r6, #4
	bne no_bullet_shot
	cmp r12, #1
	blt no_bullet_shot
	ldrh r6, [r8, #PIXMAP_X] //x coord of bullet being shot
	ldr r4, =SimplePix
	ldrh r5, [r4, #PIXMAP_Y] // y coord of player
	strh r5, [r8, #PIXMAP_Y] //store y coord of player to bullet
	// being shot so bullet shots from y coord of player
	ldr r5, =300
	strh r5, [r8, #PIXMAP_X]
	add r8, r8, #14
	cmp r8, r7
	blt continue_shooting
	ldr r8, =Bullets
continue_shooting:
	mov r12, #0
no_bullet_shot:
	pop {r4, r5, r6, r7, r9, r10, r11, r12, pc}
	


MoveSprite:
	push {r4, r5, r6, r7, lr}
	// Function accepts two inputs, x and y and the pixmap you want to use
	// This will be by how much the sprite moves in the x and y direction
	// If either value is negative, then they move to the left or up
	// If the value is positive, they move to the right and down
	mov r6, r0 //address
	mov r4, r1 //x
	mov r5, r2 //y
	cmp r4, #0
	beq no_movement_x
move_x:
	ldrh r7, [r6, #PIXMAP_X]
	add r7, r7, r4
	strh r7, [r6, #PIXMAP_X]
no_movement_x:
	cmp r5, #0
	beq no_movement_y
move_y:
	ldrh r7, [r6, #PIXMAP_Y]
	add r7, r7, r5
	strh r7, [r6, #PIXMAP_Y]
no_movement_y:
	pop {r4, r5, r6, r7, pc}
	

	
BitBlit:
	push {r4, r5, r6, r7, r8, r9, r10, r11, lr}
	mov r4, r0 // r4 is pixmap_ptr
	ldrh r5, [r4, #PIXMAP_X] // r5 is x
	ldrh r6, [r4, #PIXMAP_Y]// r6 is y
	// r3 is also used by we wont change it so no need to move into local var
	ldrh r7, [r4, #PIXMAP_WIDTH]
	ldrh r8, [r4, #PIXMAP_HEIGHT]
	ldrh r9, [r4, #PIXMAP_TRANSPARENCY]
	ldrh r10, [r4, #PIXMAP_PIXELDATA]
	add r4, r4, #PIXMAP_PIXELDATA
	b outer_blit
outer_body_blit:
	b inner_blit	
inner_body_blit:
	mov r11, r3  //r3 is the address of which buffer we want to use
	lsl r6, r6, #10
	lsl r5, r5, #1
	add r11, r11, r6
	add r11, r11, r5 
	lsr r6, r6, #10
	lsr r5, r5, #1
	ldrh r10, [r4]
	cmp r10, r9
	beq transparent
	strh r10, [r11]
transparent:
	add r4, r4, #2
	add r5, r5, #1
	sub r7, r7, #1
inner_blit:
	cmp r7, #1
	bge inner_body_blit
	sub r8, r8, #1
	ldrh r7, [r0, #PIXMAP_WIDTH]
	ldrh r5, [r0, #PIXMAP_X]
	add r6, r6, #1
outer_blit:
	cmp r8, #1
	bge outer_body_blit
	pop {r4, r5, r6, r7, r8, r9, r10, r11, pc}
	
ClearTextBuffer:
	push {r4, r5, r6, r7, r8, r9, lr}
	// To start off we will need to iter through all x and y values possible
	mov r4, #0 // r4 is the x
	mov r5, #0 // r5 is the y
	mov r6, #0 // r6 is r5 lsl shifted
	mov r7, #0 // r7 is the conncatiation of x and y
	mov r9, #0x20 //ascii value of blank space
	b outer_cond
outer_body: //This wil iter through each row
	b inner_cond
inner_body:
	ldr r8, =TEXT_BUFFER //r8 holds address of text buffer
	lsl r6, r5, #6
	add r7, r6, r4 //add into r7 the x and shifted y
	add r8, r8, r7 //r8 now holds the address we want to store into
	strb r9, [r8]
	add r4, r4, #1 //iter x axis
inner_cond:
	cmp r4, #79 // compare x with 79 since zero indexed
	ble inner_body
	add r5, r5, #1 //iter the y axis
	mov r4, #0
outer_cond:
	cmp r5, #120 //Compares with 59 since zero indexed
	ble outer_body
	pop {r4, r5, r6, r7, r8, r9, pc}
	
DrawStr:
	//prologue
	push {r4, r5, r6, r7, r8, r9, lr}
	mov r4, r0 // r4 = x
	mov r5, r1 // r5 = y
	mov r6, r2 // r6 = s
	//Now we need to loop through the string, which begins
	//at address stored in s (r6)
	b while_cond
while_body:
	//First we need to calulate the address of x and y in terms
	// of the base address x9000000
	//But first we need to check that x and y aren't outside the edges
	// so we need to check if x is bigger than 59 and if 
	// y is bigger than 79 and if so clip back to zero
	//Now we need to add y and x into the base address xc9000000
	lsl r8, r5, #6
	add r8, r8, r4 // r8= {y<5:0>, x<6:0>} 
	ldr r9, =0xc9000000
	add r9, r8
	strb r7, [r9] //Store into the address the string btye
	add r4, r4, #1 //For text you increment x 
	add r6, r6, #1 //increment r6 to get next btye of string
while_cond:
	ldrb r7, [r6]
	cmp r7, #0
	bne while_body
	pop {r4, r5, r6, r7, r8, r9, pc}
	
DrawNum:
	push {r4, r5, r6, r7, r8, r9, r10, lr}
	mov r4, r0 // r4 is x
	mov r5, r1 // r5 is y
	mov r6, r2 // r6 is the signed integer
	mov r7, r6 // r7 is copy of signed integer we can change
	mov r9, #0 // This will keep track of iter
	mov r10, #0// This will be our string of numbers in ascii
	b outer_cond_num
outer_body_num:
	mov r0, #0 // r0 is quot
	b div_cond
div_body:
	sub r7, r7, #10 // r7 is remainder
	add r0, r0, #1 // r0 is quot
div_cond:
	cmp r7, #10
	bge div_body
	add r7, r7, #48 // make the remainder into ascii by adding 0x30 or 48 in decimal
	lsl r10, r10, #8 
	add r10, r10, r7
	mov r7, r0
outer_cond_num:
	cmp r7, #0
	bgt outer_body_num
	ldr r9, =0x00500000
	str r10, [r9]
	mov r0, r4
	mov r1, r5
	mov r2, r9
	bl DrawStr
	pop {r4, r5, r6, r7, r8, r9, r10, pc}



	
	
.data
SimplePix:
	.hword 300, 120, 8, 8, 0xfffe
	.hword 0xffff, 0xffff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xffff, 0xffff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xfffe, 0xfffe, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xfffe, 0xfffe, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xfffe, 0xfffe, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xfffe, 0xfffe, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xfffe, 0xfffe, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.hword 0xfffe, 0xfffe, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff, 0x07ff
	.align 2
	

Zombies:
	//Zombie 1
	.hword 0, 20, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 2
	.hword 0, 30, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 3
	.hword 0, 40, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 4
	.hword 0, 35, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 5
	.hword 0, 25, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 6
	.hword 0, 45, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 7
	.hword 0, 50, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 8
	.hword 0, 55, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 9
	.hword 0, 60, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 10
	.hword 0, 65, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 11
	.hword 0, 70, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 12
	.hword 0, 75, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 13
	.hword 00, 80, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 14
	.hword 0, 85, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 15
	.hword 0, 90, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 16
	.hword 0, 95, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 17
	.hword 0, 100, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 18
	.hword 0, 105, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 19
	.hword 0, 110, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 20
	.hword 0, 115, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 21
	.hword 0, 120, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 22
	.hword 0, 125, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 23
	.hword 0, 130, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 24
	.hword 0, 135, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 25
	.hword 0, 140, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 26
	.hword 0, 145, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 27
	.hword 0, 150, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 28
	.hword 0, 155, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 29
	.hword 0, 160, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 30
	.hword 0, 165, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 31
	.hword 0, 170, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 32
	.hword 0, 175, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 33
	.hword 0, 180, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 34
	.hword 0, 185, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 35
	.hword 0, 190, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 36
	.hword 0, 195, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 37
	.hword 0, 200, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 38
	.hword 0, 205, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 39
	.hword 0, 210, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 40
	.hword 0, 215, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 41
	.hword 0, 220, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 42
	.hword 12, 222, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 43
	.hword 8, 221, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 44
	.hword 12, 200, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 45
	.hword 4, 23, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 46
	.hword 4, 27, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 47
	.hword 4, 33, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 48
	.hword 4, 37, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 49
	.hword 4, 43, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 50
	.hword 4, 47, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 51
	.hword 4, 53, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 52
	.hword 4, 57, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 53
	.hword 4, 63, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 54
	.hword 4, 67, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 55
	.hword 4, 73, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 56
	.hword 4, 77, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 57
	.hword 4, 83, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 58
	.hword 4, 87, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 59
	.hword 4, 93, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 60
	.hword 4, 97, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 61
	.hword 4, 103, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 62
	.hword 4, 107, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 63
	.hword 4, 113, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 64
	.hword 4, 117, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 65
	.hword 4, 123, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 66
	.hword 4, 127, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 67
	.hword 4, 133, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 68
	.hword 4, 137, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 69
	.hword 4, 143, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 70
	.hword 4, 147, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 71
	.hword 4, 153, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 72
	.hword 4, 157, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 73
	.hword 4, 163, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 74
	.hword 4, 167, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 75
	.hword 4, 173, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 76
	.hword 4, 177, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 77
	.hword 4, 183, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 78
	.hword 4, 187, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 79
	.hword 4, 193, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 80
	.hword 4, 197, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 81
	.hword 4, 203, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 82
	.hword 4, 207, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 83
	.hword 4, 213, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 84
	.hword 4, 217, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 85
	.hword 8, 222, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 86
	.hword 10, 221, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 87
	.hword 4, 23, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 88
	.hword 8, 20, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 89
	.hword 8, 143, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 90
	.hword 8, 147, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 91
	.hword 8, 153, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 92
	.hword 8, 157, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 93
	.hword 8, 163, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 94
	.hword 8, 167, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 95
	.hword 8, 173, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 96
	.hword 8, 177, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 97
	.hword 8, 183, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 98
	.hword 8, 187, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 99
	.hword 8, 193, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 100
	.hword 8, 197, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 101
	.hword 12, 20, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 102
	.hword 12, 30, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 103
	.hword 12, 40, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 104
	.hword 12, 35, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 105
	.hword 12, 25, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 106
	.hword 12, 45, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 107
	.hword 12, 50, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 108
	.hword 12, 55, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 109
	.hword 12, 60, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 110
	.hword 12, 65, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 111
	.hword 12, 70, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	//Zombie 112
	.hword 12, 75, 3, 3, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	.hword 0x07fb, 0x07fb, 0x0000
	.hword 0x07fb, 0x07fb, 0x07fb
	//Zombie 113
	.hword 12, 80, 3, 3, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	.hword 0xc89e, 0xc89e, 0x0000
	.hword 0xc89e, 0xc89e, 0xc89e
	//Zombie 114
	.hword 12, 85, 3, 3, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	.hword 0x1783, 0x1783, 0x0000
	.hword 0x1783, 0x1783, 0x1783
	//Zombie 115
	.hword 12, 90, 3, 3, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	.hword 0x7f34, 0x7f34, 0x0000
	.hword 0x7f34, 0x7f34, 0x7f34
	//Zombie 116
	.hword 12, 95, 3, 3, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	.hword 0x8bdc, 0x8bdc, 0x0000
	.hword 0x8bdc, 0x8bdc, 0x8bdc
	//Zombie 117
	.hword 12, 100, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 118
	.hword 12, 105, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 119
	.hword 12, 110, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 120
	.hword 12, 115, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 121
	.hword 12, 120, 3, 3, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	.hword 0xffff, 0xffff, 0x0000
	.hword 0xffff, 0xffff, 0xffff
	//Zombie 122
	.hword 12, 125, 3, 3, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	.hword 0x07E0, 0x07E0, 0x0000
	.hword 0x07E0, 0x07E0, 0x07E0
	//Zombie 123
	.hword 12, 130, 3, 3, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	.hword 0xa0aa, 0xa0aa, 0x0000
	.hword 0xa0aa, 0xa0aa, 0xa0aa
	//Zombie 124
	.hword 12, 135, 3, 3, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	.hword 0x8264, 0x8264, 0x0000
	.hword 0x8264, 0x8264, 0x8264
	//Zombie 125
	.hword 12, 140, 3, 3, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.hword 0xfdc0, 0xfdc0, 0x0000
	.hword 0xfdc0, 0xfdc0, 0xfdc0
	.align 2
	
Bullets:
	//Bullet 1
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 2
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 3
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 4
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 5
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 6
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 7
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 8
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 9
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 10
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 11
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 12
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 13
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 14
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 15
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 16
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 17
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 18
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 19
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 20
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 21
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 22
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 23
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 24
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 25
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 26
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 27
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 28
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 29
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 30
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 31
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 32
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 33
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 34
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 35
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 36
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 37
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 38
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 39
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 40
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 41
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 42
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 43
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 44
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 45
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 46
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 47
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 48
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 49
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	//Bullet 50
	.hword 321, 120, 2, 1, 0x0000
	.hword 0xffff, 0xffff
	.align 2
	
Hearts:
	//Heart 1
	.hword 280, 10, 5, 6, 0x0000
	.hword 0x0000, 0xf800, 0x0000, 0xf800, 0x0000
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0x0000, 0xf800, 0xf800, 0xf800, 0x0000
	.hword 0x0000, 0x0000, 0xf800, 0x0000, 0x0000
	//Heart 2
	.hword 286, 10, 5, 6, 0x0000
	.hword 0x0000, 0xf800, 0x0000, 0xf800, 0x0000
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0x0000, 0xf800, 0xf800, 0xf800, 0x0000
	.hword 0x0000, 0x0000, 0xf800, 0x0000, 0x0000
	//Heart 3
	.hword 292, 10, 5, 6, 0x0000
	.hword 0x0000, 0xf800, 0x0000, 0xf800, 0x0000
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0x0000, 0xf800, 0xf800, 0xf800, 0x0000
	.hword 0x0000, 0x0000, 0xf800, 0x0000, 0x0000
	//Heart 4
	.hword 298, 10, 5, 6, 0x0000
	.hword 0x0000, 0xf800, 0x0000, 0xf800, 0x0000
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0x0000, 0xf800, 0xf800, 0xf800, 0x0000
	.hword 0x0000, 0x0000, 0xf800, 0x0000, 0x0000
	//Heart 5
	.hword 304, 10, 5, 6, 0x0000
	.hword 0x0000, 0xf800, 0x0000, 0xf800, 0x0000
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0xf800, 0xf800, 0xf800, 0xf800, 0xf800
	.hword 0x0000, 0xf800, 0xf800, 0xf800, 0x0000
	.hword 0x0000, 0x0000, 0xf800, 0x0000, 0x0000
	.align 2
	
	
Velocity_of_Zombies:
	.hword 1
	.align 2
	
Score:
	.hword 0
	.align 2
	
back_buff:
	.skip 100000
	
Main_Menu_Message:
	.string "LEFT TO DIE"
	
Main_Menu_Buttons:
	.string "Press Button 4 to Play...."
	
How_To_Play:
	.string "Controls \n Button 1: Move Up \n Button 2: Move Down \n Button 3: Shoot the Zombies"
	
Power_Up_Message:
	.string "Button 4 during the game will buy the powerup and slow the zombies down"

Game_Over_Message:
	.string "YOU HAVE DIED"
	
ScoreStr:
	.string "Score: "
	
