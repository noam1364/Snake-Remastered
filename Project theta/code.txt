.486
.model flat,stdcall
option casemap:none

include \MASM32\INCLUDE\windows.inc 
include \MASM32\INCLUDE\masm32.inc 
include \MASM32\INCLUDE\gdi32.inc 
include \MASM32\INCLUDE\user32.inc 
include \MASM32\INCLUDE\kernel32.inc 
include \masm32\include\msvcrt.inc
include \masm32\include\winmm.inc

includelib \MASM32\LIB\masm32.lib 
includelib \MASM32\LIB\gdi32.lib 
includelib \MASM32\LIB\user32.lib 
includelib \MASM32\LIB\kernel32.lib 
includelib msvcrt.lib
includelib \masm32\lib\winmm.lib 

include drd.inc
includelib drd.lib

.data

	time SYSTEMTIME {}

	w_width dword 700				;the width and height of the window opened
	w_height dword 700

	window_name byte "Snake remastered",0

	background_off byte "game files\background.bmp",0
	background_obj Img <0,0,0,0>

	pause_menu_off byte "game files\menus\pause_menu.bmp",0
	pause_menu_obj Img <0,0,0,0>

	main_menu_off byte "game files\menus\start_menu.bmp",0
	main_menu_obj Img <0,0,0,0>

	arrow_off byte "game files\arrow.bmp",0
	arrow_obj Img <0,0,0,0>

	instructions_off byte "game files\menus\instructions.bmp",0
	instructions_obj Img <0,0,0,0>

	diff_menu_off byte "game files\menus\diff_menu.bmp",0
	diff_menu_obj Img <0,0,0,0>

	pause_off byte "game files\menus\pause_menu.bmp",0
	pause_obj Img <0,0,0,0>

	game_over_off byte "game files\menus\game_over.bmp",0		
	game_over_obj Img <0,0,0,0>

	head_right_off byte "game files\snake body\head_right.bmp",0
	head_right_obj Img <0,0,0,0>

	head_left_off byte "game files\snake body\head_left.bmp",0
	head_left_obj Img <0,0,0,0>

	head_up_off byte "game files\snake body\head_up.bmp",0
	head_up_obj Img <0,0,0,0>

	head_down_off byte "game files\snake body\head_down.bmp",0
	head_down_obj Img <0,0,0,0>

	body_off byte "game files\snake body\body.bmp",0
	body_obj Img <0,0,0,0>

	food_off byte "game files\food.bmp",0
	food_obj Img <0,0,0,0>

	zero_off byte "game files\digits\0.bmp",0
	zero_obj Img <0,0,0,0>

	one_off byte "game files\digits\1.bmp",0
	one_obj Img <0,0,0,0>

	two_off byte "game files\digits\2.bmp",0
	two_obj Img <0,0,0,0>

	three_off byte "game files\digits\3.bmp",0
	three_obj Img <0,0,0,0>

	four_off byte "game files\digits\4.bmp",0
	four_obj Img <0,0,0,0>

	five_off byte "game files\digits\5.bmp",0
	five_obj Img <0,0,0,0>

	six_off byte "game files\digits\6.bmp",0
	six_obj Img <0,0,0,0>

	seven_off byte "game files\digits\7.bmp",0
	seven_obj Img <0,0,0,0>

	eight_off byte "game files\digits\8.bmp",0
	eight_obj Img <0,0,0,0>

	nine_off byte "game files\digits\9.bmp",0
	nine_obj Img <0,0,0,0>

	mute_off byte "game files\mute.bmp",0
	mute_obj Img <0,0,0,0>

	menu_effect byte "game files\audio\menu_effect",0
	soundtrack byte "game files\audio\soundtrack.wav",0

	r_limit dword 680				;sets the border of the screen on each side(right,left,up or down)
	l_limit dword 0								
	u_limit dword 0				
	d_limit dword 580			
	
	x_pos dword 1050 dup(340)		;an array with x location of each snake link
	y_pos dword 1050 dup(380)		;an array with y location of each snake link

	arrow_x dword 530				;530=const
	arrow_y dword 300				

	menu_slot1 dword 300
	menu_slot2 dword 370
	menu_slot3 dword 440

	step dword 20					;the amount of pixels the snake moves in one step
	dirx dword 1					;the direction on the x axis
	diry dword 0					;the direction on the y axis

	speed byte 0					;sets how much dialation there is between snake movment,in milisecons
	easy byte 100
	medium byte 80
	hard byte 60

	game_on dword 2					;0=game over,1=game is on,2=main menu,3=pause

	random_x dword 0
	random_y dword 0

	body_size dword 0

	thous_x dword 200
	hunds_x dword 230
	tens_x dword 260
	units_x dword 290	
	score_y dword 35

	delay byte 100					;sets the delay between screen switches in the menus
	sound byte 1					;0=mute,1=sound is on

	milliseconds word 0
	seconds word 0

	counter byte 0
	flip_generator byte 0
	
	.code

;macro setup: 
X macro args:VARARG
	asm_txt TEXTEQU <>
	FORC char,<&args>
		IFDIF <&char>,<!\>
			asm_txt CATSTR asm_txt,<&char>
		ELSE
			asm_txt
			asm_txt TEXTEQU <>
		ENDIF
	ENDM
	asm_txt
endm

																		;ACTUALL CODE STARTS HERE:

	initiate PROC		;initiates all requiered variabels and runs all one time processes 

	pusha
		invoke drd_init, w_width, w_height,INIT_WINDOW
		invoke drd_imageLoadFile ,offset background_off,offset background_obj
		invoke drd_imageLoadFile ,offset pause_menu_off,offset pause_menu_obj
		invoke drd_imageLoadFile ,offset body_off,offset body_obj
		invoke drd_imageLoadFile ,offset food_off,offset food_obj
		invoke drd_imageLoadFile ,offset arrow_off,offset arrow_obj
		
		invoke drd_imageLoadFile ,offset diff_menu_off,offset diff_menu_obj
		invoke drd_imageLoadFile ,offset instructions_off,offset instructions_obj
		invoke drd_imageLoadFile ,offset game_over_off,offset game_over_obj
		invoke drd_imageLoadFile ,offset main_menu_off,offset main_menu_obj
		invoke drd_imageLoadFile ,offset pause_off,offset pause_obj
		
		invoke drd_imageLoadFile ,offset mute_off,offset mute_obj
		invoke drd_imageLoadFile ,offset head_right_off,offset head_right_obj
		invoke drd_imageLoadFile ,offset head_left_off,offset head_left_obj
		invoke drd_imageLoadFile ,offset head_up_off,offset head_up_obj
		invoke drd_imageLoadFile ,offset head_down_off,offset head_down_obj

		invoke drd_imageLoadFile ,offset zero_off,offset zero_obj
		invoke drd_imageLoadFile ,offset one_off,offset one_obj
		invoke drd_imageLoadFile ,offset two_off,offset two_obj
		invoke drd_imageLoadFile ,offset three_off,offset three_obj
		invoke drd_imageLoadFile ,offset four_off,offset four_obj
		
		invoke drd_imageLoadFile ,offset five_off,offset five_obj
		invoke drd_imageLoadFile ,offset six_off,offset six_obj
		invoke drd_imageLoadFile ,offset seven_off,offset seven_obj
		invoke drd_imageLoadFile ,offset eight_off,offset eight_obj
		invoke drd_imageLoadFile ,offset nine_off,offset nine_obj

		invoke drd_imageSetTransparent ,offset pause_menu_obj,0FFFF00h
		invoke drd_imageSetTransparent ,offset arrow_obj,000000h
		invoke drd_imageSetTransparent ,offset head_right_obj,0FFFFFFh
		invoke drd_imageSetTransparent ,offset head_left_obj,0FFFFFFh
		invoke drd_imageSetTransparent ,offset head_up_obj,0FFFFFFh
		invoke drd_imageSetTransparent ,offset head_down_obj,0FFFFFFh
		invoke drd_imageSetTransparent ,offset body_obj,0FFFFFFh
		invoke drd_imageSetTransparent ,offset food_obj,0FFFFFFh

		X	mov eax,w_width \ sub eax,body_obj.iwidth \ mov r_limit,eax
		X	xor eax,eax \ mov l_limit,eax
		X	mov eax,w_height \ sub eax,body_obj.iheight \ mov d_limit,eax
		X	mov eax,100 \ mov u_limit,eax

	popa
	ret

	initiate ENDP


	score_system PROC		;displayes the current score

	pusha
		X	cmp game_on,0 \ jg in_game \ jmp game_over_screen	;changes the cords in which the score will be displayed,accourding to
		in_game:												;the status of the game (game over screen\in game)
		X	mov thous_x,200 \ mov hunds_x,230
		X	mov tens_x,260 \ mov units_x,290
		mov score_y,35
		jmp thousands

		game_over_screen:
		X	mov thous_x,530 \ mov hunds_x,560
		X	mov tens_x,590 \ mov units_x,620 
		mov score_y,535

		thousands:
		X	mov eax,body_size \ mov ebx,100 \ div ebx \ mov ebx,10 \ xor edx,edx \ div ebx   
		X	cmp eax,0 \ je zero 
		X	cmp eax,1 \ je one
		X	cmp eax,2 \ je two
		X	cmp eax,3 \ je three
		X	cmp eax,4 \ je four 
		X	cmp eax,5 \ je five
		X	cmp eax,6 \ je six
		X	cmp eax,7 \ je seven
		X	cmp eax,8 \ je eight
		X	cmp eax,9 \ je nine

		hundreds:
		X	mov eax,body_size \ mov ebx,100 \ xor edx,edx \ div ebx \ mov ebx,10 \ xor edx,edx \ div ebx
		X	cmp edx,0 \ je zero1 
		X	cmp edx,1 \ je one1
		X	cmp edx,2 \ je two1
		X	cmp edx,3 \ je three1
		X	cmp edx,4 \ je four1 
		X	cmp edx,5 \ je five1
		X	cmp edx,6 \ je six1
		X	cmp edx,7 \ je seven1
		X	cmp edx,8 \ je eight1
		X	cmp edx,9 \ je nine1

		tens:
		X	mov eax,body_size \ mov ebx,100 \ xor edx,edx \ div ebx \ mov eax,edx \ xor edx,edx \ mov ebx,10 \ div ebx
		X	cmp eax,0 \ je zero2
		X	cmp eax,1 \ je one2
		X	cmp eax,2 \ je two2
		X	cmp eax,3 \ je three2
		X	cmp eax,4 \ je four2 
		X	cmp eax,5 \ je five2
		X	cmp eax,6 \ je six2
		X	cmp eax,7 \ je seven2
		X	cmp eax,8 \ je eight2
		X	cmp eax,9 \ je nine2
		
		units:
		X	mov eax,body_size \ mov ebx,10 \ xor edx,edx \ div ebx 
		X	cmp edx,0 \ je zero3 
		X	cmp edx,1 \ je one3
		X	cmp edx,2 \ je two3
		X	cmp edx,3 \ je three3
		X	cmp edx,4 \ je four3
		X	cmp edx,5 \ je five3
		X	cmp edx,6 \ je six3
		X	cmp edx,7 \ je seven3
		X	cmp edx,8 \ je eight3
		X	cmp edx,9 \ je nine3

		zero:
		X	invoke drd_imageDraw ,offset zero_obj,thous_x,score_y \ jmp hundreds
		one:
		X	invoke drd_imageDraw ,offset one_obj,thous_x,score_y \ jmp hundreds
		two:
		X	invoke drd_imageDraw ,offset two_obj,thous_x,score_y \ jmp hundreds
		three:
		X	invoke drd_imageDraw ,offset three_obj,thous_x,score_y \ jmp hundreds
		four:
		X	invoke drd_imageDraw ,offset four_obj,thous_x,score_y \ jmp hundreds
		five:
		X	invoke drd_imageDraw ,offset five_obj,thous_x,score_y \ jmp hundreds
		six:
		X	invoke drd_imageDraw ,offset six_obj,thous_x,score_y \ jmp hundreds
		seven:
		X	invoke drd_imageDraw ,offset seven_obj,thous_x,score_y \ jmp hundreds
		eight:
		X	invoke drd_imageDraw ,offset eight_obj,thous_x,score_y \ jmp hundreds
		nine:
		X	invoke drd_imageDraw ,offset nine_obj,thous_x,score_y \ jmp hundreds


		zero1:
		X	invoke drd_imageDraw ,offset zero_obj,hunds_x,score_y \ jmp tens
		one1:
		X	invoke drd_imageDraw ,offset one_obj,hunds_x,score_y \ jmp tens
		two1:
		X	invoke drd_imageDraw ,offset two_obj,hunds_x,score_y \ jmp tens
		three1:
		X	invoke drd_imageDraw ,offset three_obj,hunds_x,score_y \ jmp tens
		four1:
		X	invoke drd_imageDraw ,offset four_obj,hunds_x,score_y \ jmp tens
		five1:
		X	invoke drd_imageDraw ,offset five_obj,hunds_x,score_y \ jmp tens
		six1:
		X	invoke drd_imageDraw ,offset six_obj,hunds_x,score_y \ jmp tens
		seven1:
		X	invoke drd_imageDraw ,offset seven_obj,hunds_x,score_y \ jmp tens
		eight1:
		X	invoke drd_imageDraw ,offset eight_obj,hunds_x,score_y \ jmp tens
		nine1:
		X	invoke drd_imageDraw ,offset nine_obj,hunds_x,score_y \ jmp tens

		zero2:
		X	invoke drd_imageDraw ,offset zero_obj,tens_x,score_y \ jmp units
		one2:
		X	invoke drd_imageDraw ,offset one_obj,tens_x,score_y \ jmp units
		two2:
		X	invoke drd_imageDraw ,offset two_obj,tens_x,score_y \ jmp units
		three2:
		X	invoke drd_imageDraw ,offset three_obj,tens_x,score_y \ jmp units
		four2:
		X	invoke drd_imageDraw ,offset four_obj,tens_x,score_y \ jmp units
		five2:
		X	invoke drd_imageDraw ,offset five_obj,tens_x,score_y \ jmp units
		six2:
		X	invoke drd_imageDraw ,offset six_obj,tens_x,score_y\ jmp units
		seven2:
		X	invoke drd_imageDraw ,offset seven_obj,tens_x,score_y \ jmp units
		eight2:
		X	invoke drd_imageDraw ,offset eight_obj,tens_x,score_y \ jmp units
		nine2:
		X	invoke drd_imageDraw ,offset nine_obj,tens_x,score_y \ jmp units
	
		zero3:
		X	invoke drd_imageDraw ,offset zero_obj,units_x,score_y \ jmp exit
		one3:
		X	invoke drd_imageDraw ,offset one_obj,units_x,score_y \ jmp exit
		two3:
		X	invoke drd_imageDraw ,offset two_obj,units_x,score_y \ jmp exit
		three3:
		X	invoke drd_imageDraw ,offset three_obj,units_x,score_y \ jmp exit
		four3:
		X	invoke drd_imageDraw ,offset four_obj,units_x,score_y \ jmp exit
		five3:
		X	invoke drd_imageDraw ,offset five_obj,units_x,score_y \ jmp exit
		six3:
		X	invoke drd_imageDraw ,offset six_obj,units_x,score_y \ jmp exit
		seven3:
		X	invoke drd_imageDraw ,offset seven_obj,units_x,score_y \ jmp exit
		eight3:
		X	invoke drd_imageDraw ,offset eight_obj,units_x,score_y \ jmp exit
		nine3:
		X	invoke drd_imageDraw ,offset nine_obj,units_x,score_y 
		
	exit:
	popa
	ret

	score_system ENDP
	
	
	random_generator PROC

	pusha
		X	xor eax,eax \ xor edx,edx \ mov counter,0		;init the variables accourding to current system time
		invoke GetSystemTime ,addr time
		X	mov ax, time.wMilliseconds \ mov milliseconds,ax
		X	mov ax,time.wSecond \ mov seconds,ax

		start:		;multiplis the current millisecond by 11,then divides by 16 and rounds the result up to by a multiplie of 20
		xor eax,eax
		mov ax,milliseconds
		X	mov ebx,11 \ mov ecx,16
		X	mul ebx \ div ecx
		xor edx,edx
		X	div step \ mul step 
		mov random_x,eax 

		xor eax,eax
		X	mov ax,seconds \ mul milliseconds \ mov ebx,8 \ mov ecx,84
		X	div ebx \ mov ebx,edx \ xor edx,edx
		X	mov ax,seconds \ mul milliseconds \ div ecx \ mov eax,edx 
		X	mul ebx \ div step \ mul step 		

		X	cmp flip_generator,1 \ je flip_randomY
		X	mov random_y,eax \ mov flip_generator,1 \ add random_y,100 \ jmp check_spawn_currect
		flip_randomY:
		X	mov flip_generator,0 \ mov ebx,580 \ sub ebx,eax \ mov random_y,ebx				;the random Ycord generated is still located in EAX
		add random_y,100

		check_spawn_currect:
		X	cmp counter,34 \ je exit		;checks if the apple spawned inside the snakes body,and if so,increases the seedes and generate a new cords
		mov esi,body_size
		check_loop:	
		X	cmp esi,0 \ je exit
		X	dec esi	\ mov eax,random_x \ cmp eax,x_pos[esi*4] \ jne check_loop		;body_size's value is at lease one
		X	mov eax,random_y \ cmp eax,y_pos[esi*4] \ jne check_loop 

		X	add milliseconds,30 \ add seconds,2 \ inc counter
		X	cmp milliseconds,999 \ jg init_x
		X	cmp seconds,59 \ jg init_y
		jmp start
		init_x:
		mov milliseconds,0 
		X	cmp seconds,59 \ jg init_y \ jmp start
		init_y:
		X	mov seconds,0 \ jmp start

	exit:	
	popa
	ret

	random_generator ENDP


	start_menu PROC

	pusha
		main_menu:
		invoke drd_imageDraw ,offset main_menu_obj,0,0
		invoke drd_imageDraw ,offset arrow_obj,arrow_x,arrow_y
		X	cmp sound,1 \ je continue2
		invoke drd_imageDraw ,offset mute_obj,615,15
		continue2:
		
		invoke drd_processMessages
		invoke drd_flip
		
		invoke GetAsyncKeyState, VK_ESCAPE			;delayed false input prevention
		invoke Sleep,delay
		X	invoke GetAsyncKeyState, VK_LSHIFT \ cmp eax,0 \ jne check_mute_main
		X	invoke GetAsyncKeyState, VK_SPACE \ cmp eax,0 \ jne switch_menu
		X	invoke GetAsyncKeyState, VK_RETURN \ cmp eax,0 \ jne switch_menu
		X	invoke GetAsyncKeyState, VK_UP \ cmp eax,0 \ jne arrow_up_main
		X	invoke GetAsyncKeyState, 57h \ cmp eax,0 \ jne arrow_up_main
		X	invoke GetAsyncKeyState, VK_DOWN \ cmp eax,0 \ jne arrow_down_main
		X	invoke GetAsyncKeyState, 53h \ cmp eax,0 \ jne arrow_down_main
		jmp main_menu

		inst_menu:
		invoke drd_processMessages
		invoke drd_flip
		invoke drd_imageDraw ,offset instructions_obj,0,0 
		X	cmp sound,1 \ je continue3
		invoke drd_imageDraw ,offset mute_obj,615,15
		
		continue3:
		invoke GetAsyncKeyState, VK_SPACE			;delayed false input prevention
		invoke GetAsyncKeyState, VK_RETURN
		invoke GetAsyncKeyState, VK_UP
		invoke GetAsyncKeyState, 57h	
		invoke GetAsyncKeyState, VK_DOWN
		invoke GetAsyncKeyState, 53h	
		invoke Sleep,delay
		X	invoke GetAsyncKeyState, VK_LSHIFT \ cmp eax,0 \ jne check_mute_inst
		X	invoke GetAsyncKeyState, VK_ESCAPE \ cmp eax,0 \ je inst_menu
		X	cmp sound,0 \ je exit_to_main_mute2
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		exit_to_main_mute2:
		invoke Sleep,delay
		jmp main_menu

		diff_menu:
		invoke drd_processMessages
		invoke drd_flip
		invoke drd_imageDraw ,offset diff_menu_obj,0,0
		invoke drd_imageDraw ,offset arrow_obj,arrow_x,arrow_y
		X	cmp sound,1 \ je continue4
		invoke drd_imageDraw ,offset mute_obj,615,15
		
		continue4:
		X	invoke GetAsyncKeyState, VK_ESCAPE \ cmp eax,0 \ je continue
		X	cmp sound,0 \ je exit_to_main_mute
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		exit_to_main_mute:
		X	mov eax,menu_slot1 \ mov arrow_y,eax \ jmp main_menu
		continue:
		invoke Sleep,delay
		X	invoke GetAsyncKeyState, VK_LSHIFT \ cmp eax,0 \ jne check_mute_diff
		X	invoke GetAsyncKeyState, VK_UP \ cmp eax,0 \ jne arrow_up_diff_menu
		X	invoke GetAsyncKeyState, 57h \ cmp eax,0 \ jne arrow_up_diff_menu
		X	invoke GetAsyncKeyState, VK_DOWN \ cmp eax,0 \ jne arrow_down_diff_menu
		X	invoke GetAsyncKeyState, 53h\ cmp eax,0 \ jne arrow_down_diff_menu
		X	invoke GetAsyncKeyState, VK_SPACE \ cmp eax,0 \ jne set_diff
		X	invoke GetAsyncKeyState, VK_RETURN \ cmp eax,0 \ jne set_diff
		jmp diff_menu

		switch_menu:
		X	cmp sound,0 \ je switch_with_mute
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		switch_with_mute:
		invoke Sleep,delay
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je diff_menu
		jmp inst_menu

		arrow_up_main:
		X	cmp sound,0 \ je no_click1
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		no_click1:
		invoke Sleep,delay
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je main_menu
		X	mov arrow_y,eax \ jmp main_menu

		arrow_down_main:
		X	cmp sound,0 \ je no_click2
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		no_click2:
		invoke Sleep,delay
		X	mov eax,menu_slot2 \ cmp arrow_y,eax \ je main_menu
		X	mov arrow_y,eax \ jmp main_menu

		arrow_up_diff_menu:
		X	cmp sound,0 \ je no_click3
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		no_click3:
		invoke GetAsyncKeyState, VK_SPACE		;false delayed input prevention
		invoke GetAsyncKeyState, VK_RETURN	
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je diff_menu
		X	sub arrow_y,70 \ jmp diff_menu							;one slot up
		
		arrow_down_diff_menu:
		X	cmp sound,0 \ je no_click4
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		no_click4:
		invoke GetAsyncKeyState, VK_SPACE		;false delayed input prevention
		invoke GetAsyncKeyState, VK_RETURN
		X	mov eax,menu_slot3 \ cmp arrow_y,eax \ je diff_menu
		X	add arrow_y,70 \ jmp diff_menu							;one slot down

		check_mute_main:
		X	cmp sound,1 \ je mute_sfx_main
		X	mov sound,1 \ jmp main_menu
		mute_sfx_main:
		X	mov sound,0 \ jmp main_menu

		check_mute_inst:
		X	cmp sound,1 \ je mute_sfx_inst
		X	mov sound,1 \ jmp inst_menu
		mute_sfx_inst:
		X	mov sound,0 \ jmp inst_menu

		check_mute_diff:
		X	cmp sound,1 \ je mute_sfx_diff
		X	mov sound,1 \ jmp diff_menu
		mute_sfx_diff:
		X	mov sound,0 \ jmp diff_menu
		
		set_diff:
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je set_easy
		X	mov eax,menu_slot2 \ cmp arrow_y,eax \ je set_medium
		jmp set_hard
		set_easy:
		X	mov al,easy \ mov speed,al \ jmp start_game
		set_medium:
		X	mov al,medium \ mov speed,al \ jmp start_game
		set_hard:
		X	mov al,hard \ mov speed,al
	
		start_game:
		X	mov x_pos[0],340 \ mov y_pos[0],380
		X	mov eax,menu_slot1 \ mov arrow_y,eax
		mov game_on,1
		invoke random_generator							;generates the first food
		X	cmp sound,0 \ je start_without_sound
		invoke PlaySound,offset soundtrack,NULL,SND_ASYNC
		start_without_sound:
		X	mov body_size,0
		invoke Sleep,1300
		
	exit:			
	invoke GetAsyncKeyState, VK_ESCAPE			;false delayed input prevention:	 
	invoke GetAsyncKeyState, VK_LSHIFT			
	invoke GetAsyncKeyState, VK_RIGHT
	invoke GetAsyncKeyState, 44h
	invoke GetAsyncKeyState, VK_LEFT
	invoke GetAsyncKeyState, 41h 
	invoke GetAsyncKeyState, VK_UP 
	invoke GetAsyncKeyState, 57h
	invoke GetAsyncKeyState, VK_DOWN
	invoke GetAsyncKeyState, 53h

	popa
	ret
	start_menu ENDP


	draw_objects PROC

	pusha
		X	cmp game_on,1 \ je in_game  
		X	cmp game_on,3 \ je pause_menu 
		jmp gameOver			;if game_on==0

		pause_menu:
		X	invoke drd_imageDraw ,offset pause_menu_obj,0,0
		X	invoke drd_imageDraw ,offset arrow_obj,arrow_x,arrow_y
		jmp check_mute
				
		in_game:
		invoke drd_imageDraw ,offset background_obj,0,0 
		invoke drd_imageDraw ,offset food_obj,random_x,random_y

		mov esi,body_size
		X	cmp dirx,1 \ je face_right
		X	cmp dirx,-1 \ je face_left
		X	cmp diry,-1 \ je face_up
		X	cmp diry,1 \ je face_down
		
		face_right:
		invoke drd_imageDraw ,offset head_right_obj,x_pos[0],y_pos[0]
		jmp draw_body
		
		face_left:
		invoke drd_imageDraw ,offset head_left_obj,x_pos[0],y_pos[0]
		jmp draw_body
		
		face_up:
		invoke drd_imageDraw ,offset head_up_obj,x_pos[0],y_pos[0]
		jmp draw_body
		
		face_down:
		invoke drd_imageDraw ,offset head_down_obj,x_pos[0],y_pos[0]
		;jmp draw_body

		draw_body: 
		X	cmp esi,0 \ je check_mute
		invoke drd_imageDraw ,offset body_obj,x_pos[esi*4],y_pos[esi*4]
		X	dec esi \ jmp draw_body  

		gameOver:
		invoke drd_imageDraw ,offset game_over_obj,0,0
		invoke drd_imageDraw ,offset arrow_obj,arrow_x,arrow_y

		check_mute: 
		X	cmp sound,0 \ je draw_mute \ jmp exit
		draw_mute:
		invoke drd_imageDraw ,offset mute_obj,615,15

	exit:
	popa
	ret

	draw_objects ENDP

															;IMPORTENT SHIT STARTS HERE:
	user_input PROC		

	pusha 
		start:
		X	cmp game_on,1 \ je in_game \ cmp game_on,0 \ je game_over_menu
		pause_menu:			;if game_on==3
		invoke Sleep,delay
		X	invoke GetAsyncKeyState, VK_LSHIFT \ cmp eax, 0 \ jne change_sound 
		X	invoke GetAsyncKeyState, VK_SPACE \ cmp eax,0 \ jne check_arrow_pause
		X	invoke GetAsyncKeyState, VK_RETURN \ cmp eax,0 \ jne check_arrow_pause
		X	invoke GetAsyncKeyState, VK_UP \ cmp eax,0 \ jne arrow_up
		X	invoke GetAsyncKeyState, 57h \ cmp eax,0 \ jne arrow_up
		X	invoke GetAsyncKeyState, VK_DOWN \ cmp eax,0 \ jne arrow_down
		X	invoke GetAsyncKeyState, 53h \ cmp eax,0 \ jne arrow_down
		X	invoke GetAsyncKeyState, VK_ESCAPE								;delayed false input prevention:
		;X	invoke GetAsyncKeyState, VK_RIGHT 
		X	invoke GetAsyncKeyState, VK_LEFT
		jmp exit

		game_over_menu:
		invoke Sleep,delay
		X	invoke GetAsyncKeyState, VK_LSHIFT \ cmp eax, 0  \ jne change_sound 
		X	invoke GetAsyncKeyState, VK_SPACE \ cmp eax, 0  \ jne check_arrow_game_over
		X	invoke GetAsyncKeyState, VK_RETURN \ cmp eax, 0  \ jne check_arrow_game_over
		X	invoke GetAsyncKeyState, VK_UP \ cmp eax,0 \ jne arrow_up
		X	invoke GetAsyncKeyState, 57h \ cmp eax,0 \ jne arrow_up
		X	invoke GetAsyncKeyState, VK_DOWN \ cmp eax,0 \ jne arrow_down
		X	invoke GetAsyncKeyState, 53h \ cmp eax,0 \ jne arrow_down
		X	invoke GetAsyncKeyState, VK_ESCAPE			;delayed false input prevention:
		X	invoke GetAsyncKeyState, VK_RIGHT 
		X	invoke GetAsyncKeyState, VK_LEFT
		jmp exit
		
		in_game:
		X	invoke GetAsyncKeyState, VK_LSHIFT \ cmp eax, 0 \ jne change_sound 
		X	invoke GetAsyncKeyState, VK_ESCAPE \ cmp eax, 0  \ jne pause_game
		X	invoke GetAsyncKeyState, VK_UP \ cmp eax, 0 \ jne up 
		X	invoke GetAsyncKeyState, VK_DOWN \ cmp eax, 0 \ jne down
		X	invoke GetAsyncKeyState, VK_RIGHT \ cmp eax, 0  \ jne right
		X	invoke GetAsyncKeyState, VK_LEFT \ cmp eax, 0 \ jne left 
		X	invoke GetAsyncKeyState, 57h \ cmp eax, 0 \ jne up
		X	invoke GetAsyncKeyState, 53h \ cmp eax, 0 \ jne down
		X	invoke GetAsyncKeyState, 44h \ cmp eax, 0  \ jne right
		X	invoke GetAsyncKeyState, 41h \ cmp eax, 0 \ jne left 
		jmp exit

		right:
		X	cmp dirx,-1 \ je exit
		X	mov dirx,1 \ mov diry,0 \ jmp exit
		left:
		X	cmp dirx,1 \ je exit
		X	mov dirx,-1 \ mov diry,0 \ jmp exit
		up:
		X	cmp diry,1 \ je exit
		X	mov dirx,0 \ mov diry,-1 \ jmp exit
		down:
		X	cmp diry,-1 \ je exit
		X	mov dirx,0 \ mov diry,1 \ jmp exit

		check_arrow_pause:
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je unpause_game 
		X	mov arrow_y,eax \ invoke start_menu
		jmp start					;so when ip will point back here,it will start from the start,not from this line

		check_arrow_game_over:
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je restart 
		X	mov arrow_y,eax \ mov game_on,2 \ invoke start_menu
		jmp start					;so when ip will point back here,it will start from the start,not from this line

		arrow_up:
		X	cmp sound,0 \ je skip_sound1
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		skip_sound1:
		X	mov eax,menu_slot1 \ cmp arrow_y,eax \ je exit
		X	mov arrow_y,eax \ jmp exit

		arrow_down:
		X	cmp sound,0 \ je skip_sound2
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		skip_sound2:
		X	mov eax,menu_slot2 \ cmp arrow_y,eax \ je exit
		X	mov arrow_y,eax \ jmp exit

		restart:
		X	mov x_pos[0],340 \ mov y_pos[0],380 \ mov game_on,1
		X	mov dirx,1 \ mov diry,0 \ mov body_size,0
		invoke random_generator									;to generate new food  
		X	cmp sound,1 \ je restart_with_sound \ invoke Sleep,1300 \ jmp exit
		restart_with_sound:
		invoke PlaySound,addr soundtrack,NULL,SND_ASYNC
		invoke Sleep,1300
		jmp exit

		pause_game:
		;delayed false input prevention:
		invoke GetAsyncKeyState, VK_SPACE
		invoke GetAsyncKeyState, VK_RETURN  
		invoke GetAsyncKeyState, VK_LSHIFT 

		X	cmp sound,0 \ je skip_sound3
		invoke PlaySound,offset menu_effect,NULL,SND_ASYNC
		skip_sound3:
		X	invoke GetAsyncKeyState, VK_ESCAPE		;false delayed input prevention
		X	mov game_on,3 
		jmp exit

		unpause_game:
		mov game_on,1
		X	cmp sound,1 \ je unpause_with_sound \ invoke Sleep,1300 \ jmp exit
		unpause_with_sound:  
		invoke PlaySound,addr soundtrack,NULL,SND_ASYNC
		invoke Sleep,1300
		jmp exit

		change_sound:
		X	cmp sound,1 \ je mute_game
		X	mov sound,1 \ cmp game_on,1 \ jne exit 
		X	invoke PlaySound,addr soundtrack,NULL,SND_ASYNC \ jmp exit
		mute_game:
		invoke PlaySound,NULL,NULL,SND_ASYNC 
		X	mov sound,0 \ jmp exit
		
	exit:
	popa
	ret

	user_input ENDP
	

	movment_manager PROC
	
	pusha
		X	cmp game_on,1 \ jne exit					;to block movment while in pause
		X	mov esi,body_size \ cmp esi,0 \ je head
		body:
		X	mov eax,x_pos[esi*4-4] \ mov x_pos[esi*4],eax
		X	mov eax,y_pos[esi*4-4] \ mov y_pos[esi*4],eax
		X	dec esi \ cmp esi,0 \ jne body
		head:
		X	mov eax,dirx \ mul step \ add x_pos[0],eax
		X	mov eax,diry \ mul step \ add y_pos[0],eax
			
	exit: 
	popa
	ret

	movment_manager ENDP

	
	body_collition PROC

	pusha
		X	cmp game_on,1 \ jne exit  
		X	mov eax,x_pos[0] \ cmp eax,r_limit \ jg gg
		X	mov eax,x_pos[0] \ cmp eax,l_limit \ jl gg
		X	mov eax,y_pos[0] \ cmp eax,u_limit \ jl gg 
		X	mov eax,y_pos[0] \ cmp eax,d_limit \ jg gg

		X	mov eax,x_pos[0] \ mov ebx,y_pos[0] \ mov esi,body_size \ inc esi
		tangeled:
		X	dec esi \ cmp esi,3 \ jbe exit		;placed here rather then at the end of the label,for if body_size<=3
		X	cmp eax,x_pos[esi*4] \ jne tangeled					;to be reviewed for efficency
		X	cmp ebx,y_pos[esi*4] \ je gg 
		jmp tangeled	
		
		gg:
		X	mov eax,menu_slot1 \ mov arrow_y,eax
		mov game_on,0
		invoke PlaySound,NULL,NULL,SND_ASYNC

	exit:
	popa	
	ret

	body_collition ENDP


	food_collition PROC

	pusha
		X	mov eax,random_x \ cmp x_pos[0],eax \ jne exit
		X	mov eax,random_y \ cmp y_pos[0],eax \ jne exit
		invoke random_generator		;generate new food:
		inc body_size
		X	mov esi,body_size \ mov eax,random_x \ mov x_pos[esi*4],eax \ mov eax,random_y \ mov y_pos[esi*4],eax 
		
	exit:
	popa
	ret

	food_collition ENDP
	

	main PROC

		invoke initiate
		invoke start_menu
		again:		
		invoke drd_processMessages
		invoke drd_flip
		invoke Sleep,speed
		invoke user_input
		invoke movment_manager
		invoke body_collition
		invoke draw_objects
		invoke food_collition
		invoke score_system
		jmp again
	ret
	main ENDP
	end main