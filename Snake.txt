TITLE SnakeXenia.asm

INCLUDE Irvine32.inc
.data
	String1		BYTE "*********WELCOME TO SNAKE XENIA **********",0
	String2		BYTE "Your Score is: ",0				   	
	String3		BYTE "GAME OVER! Play Again?(Y/N)",0						
	String4		BYTE "Game speed is :",0  					
	x_head 		BYTE ?								; Variable that holds the "x" of the head of the snake.
	y_head 		BYTE ?								; Variable that holds the "y" of the head of the snake.
	head 		BYTE 2					
	node 		BYTE "#"							
	x_apple		BYTE ?								; Variable that holds the "x" of the apple.
	y_apple		BYTE ?								; Variable that holds the "y" of the apple.
	appleeaten	BYTE 0								; Apple eaten or not?
	direction 	BYTE 0  							
	olddirection 	BYTE 0								
	bricks1 	BYTE "±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±±",0
	bricks2 	BYTE "±						 	±",0
	apple		BYTE "@"							; Character for apple.
	x_tail		BYTE ?								; Variable that holds the "x" of the tail of the snake. 
	y_tail		byte ?								; Variable that holds the "x" of the tail of the snake. 
	Nodes_X		BYTE 735 DUP(0)	
	Nodes_Y		BYTE 735 DUP(0)	
	NumOfNodes	DWORD 0								; The number of nodes snake has.
	score		DWORD 0								
	speed		WORD 0
.code
main PROC
begin::
	pushad
	call Clrscr
	mov ax,90
	mov speed,ax
	mov eax,0
	mov score,eax
	mov eax,0
	mov NumOfNodes,eax
	mov al,0
	mov appleeaten,al
	popad
	mov dl, 14	
	mov dh, 2
	call Gotoxy	
	mov eax, white+(blue*16)		;Text Colour of Welcome Note
	call SetTextColor
	mov edx, OFFSET String1			;Printing Welcome Note(String1)
	call WriteString
	mov dl, 0
	mov dh, 8			
	call Gotoxy			
	mov eax, green					;Green Colour
	call SetTextColor				;Text Colour GameSpeed Note(String4)
	mov edx, OFFSET String4		
	call WriteString
	mov dl, 14			
	mov dh, 4			
	call Gotoxy			
	mov eax, cyan+(black*16)		;Background Colour
	call SetTextColor		
	mov edx, OFFSET bricks1			;Bricks(walls)
	call WriteString
	mov ah, 20
Wall:								;Wall Printing
	mov dl, 14
	mov dh, ah
	call Gotoxy	
	dec ah
	mov edx, OFFSET bricks2	
	call WriteString
	cmp ah, 4	
	jg Wall
	mov dl, 14		
	mov dh, 21
	call Gotoxy
	mov edx, OFFSET bricks1			;Printing Pattern
	call WriteString
RandomX:
	mov eax,36
	mov x_head, al
	mov esi, OFFSET Nodes_X		; bounds of the zone. Then
	mov [esi], al
	mov dl, al					; the coordinate is located into the array.
RandomY:
	mov eax,12
	mov y_head, al				; specified according to the
	mov esi, OFFSET Nodes_Y
	mov [esi], al				; bounds of the zone. Then
	mov dh, al
	call Gotoxy					; the coordinate is located into the array.
	mov al, head			
	call WriteChar
Start:
	call CrashSnake				; Control if the snake eats itself.
	call ReadKey				; Read a key from the keyboard.
	jz SameDirection			; If no key is pressed the current direction be applied.
	cmp ah, 51H					; Keys except arrows
	jg Start
	cmp ah, 47H					; are not
	jl Start
	call Move					; Start to move
	call Configure				; Some configurations about the snake.
	call PrintNodes				; Print the snake on the screen with its nodes.
	jmp Start
SameDirection:
	mov ah, direction			; The label to specify the current direction.
	call Move					; Continue moving
	call Configure				; by controlling your configuration
	call PrintNodes				; and by printing yourself on the screen.
	jmp Start
main ENDP
Move PROC USES eax edx
	mov direction, ah			; Game Speed is calculated and
	call GameSpeed				; printed on the screen.
	mov ax, speed
	movzx eax, ax				; It is done by delaying the motion
	call Delay					; of the snake.
	mov dl, 0					; The calculated speed is printed
	mov dh, 9					; on the screen.
	call Gotoxy
	mov ax, speed
	movzx eax, ax
	call WriteInt				; Here is the speed of the game.
	mov dl, x_head
	mov dh, y_head
	call Gotoxy
	mov al, ' '
	call WriteChar		
	call EatApple
	mov ah, direction			; The direction is passed to register "ah".
	mov al, olddirection		; The old direction is passed to register "al".
	cmp dl, 64					; If the head of the snake is
	jge GameOver
	cmp dl, 14					; located ob the bounds of the wall,
	jle GameOver
	cmp dh, 21					; unfortunately the game ends.
	jge GameOver
	cmp dh, 4
	jle GameOver
	cmp ah, 48H					; Control of the motion if it is upwards.
	je Up
	cmp ah, 50H					; Control of the motion if it is downwards.
	je Down
	cmp ah, 4DH					; Control of the motion if it is rightwards.
	je Right
	cmp ah, 4BH					; Control of the motion if it is leftwards.
	je Left
	jmp Finish
Up:	
	mov olddirection, 48H		; The motion of the snake is
	cmp al, 50H
	je Down						; upwards now.
	dec dh
	jmp UpdateHeadLoc
Down:
	mov olddirection, 50H		; The motion of the snake is
	cmp al, 48H
	je Up						; downwards now.
	inc dh
	jmp UpdateHeadLoc
Right:
	mov olddirection, 4DH		; The motion of the snake is
	cmp al, 4BH
	je Left						; leftwards now.
	inc dl
	jmp UpdateHeadLoc
Left:
	mov olddirection, 4BH		; The motion of the snake is
	cmp al, 4DH
	je Right					; rightwards now.
	dec dl
	jmp UpdateHeadLoc
UpdateHeadLoc:
	mov x_head, dl				; The head of the snake is
	mov y_head, dh
	call Gotoxy			; located on the updated location.
	mov al, head
	call WriteChar
Finish:
	ret
GameOver:
	mov dl, x_head				; If any error occurs the game ends
	mov dh, y_head
	call Gotoxy					; by this label.
	mov al, head
	call WriteChar				; After a short delay,
	mov eax, 1000
	call Delay					; "GAME OVER!!!" message is displayed
	mov dl, 33
	mov dh, 13					; on the screen and is returned to OS.
	call Gotoxy
	mov eax, white+(red*16)		;Set Text Color of "GAME OVER!!!"
	call SetTextColor
	mov edx, OFFSET String3		;Print Offset String3
	call WriteString
	mov dl, 20
	mov dh, 24
	pushad
	call readchar
	cmp al,'y'
	popad
	je begin
	call Gotoxy
	exit
Move ENDP
EatApple PROC USES eax edx
NewApple:
	mov al, appleeaten			; After each apple is eaten
	cmp al, 0
	jne NotEaten				; another apple is located randomly
RandomX:
	mov appleeaten, 1			; on the screen.
	mov eax, 64
	call RandomRange			; The "x" and the "y" of the new apple
	cmp al, 15
	jl RandomX					; are randomly specified and
	mov x_apple, al
	mov dl, al					; printed on the screen.
RandomY:
	mov eax, 18
	call RandomRange
	cmp al, 5
	jl RandomY
	mov y_apple, al
	mov dh, al
	call Gotoxy
	mov al, apple
	call WriteChar
	mov al, dl
NotEaten:
	mov al, x_head				; If the current apple on the screen
	mov ah, y_head
	mov dl, x_apple				; is not eaten,this means the head didn't
	mov dh, y_apple
	cmp ax, dx					; pass over the apple and eat it,another
	jne Finish
	mov eax, NumOfNodes			; apple is not produced.
	inc eax
	mov NumOfNodes, eax			; Number Of Nodes of the snake is updated.
	mov appleeaten, 0			; There's no apple eaten yet.
	call AddNodes				; Update and add nodes to the snake.
	call GameScore				; Game Score is updated after each eaten apple.
	mov dl, 30
	mov dh, 23
	call Gotoxy
	mov edx, OFFSET String2		; Game Score is printed on the screen.
	call WriteString
	mov eax, score	
	call WriteInt
	jmp Finish
Finish:
	ret
EatApple ENDP
AddNodes PROC USES eax ebx ecx esi
	mov ebx, NumOfNodes			; The procedure to control the
	cmp ebx, 1
	jge Continue				; addition of the nodes and update the 
	mov esi, OFFSET Nodes_X
	mov al, x_head				; number of nodes.
	mov [esi], al
	mov al, [esi]
	mov esi, OFFSET Nodes_Y
	mov al, y_head
	mov [esi], al
	mov al, [esi]
	jmp Finish
Continue:						; If appleeaten is "0" then
	mov al, appleeaten			; the current apple was eaten and
	movzx eax, al				; if it is "1",
	cmp al, 0					; then the apple was not eaten.
	jne NotEaten
Eaten:
	mov ecx, NumOfNodes			; Number of the nodes is
	inc ecx						; passed to register "ecx".
ShiftRight:
	mov ebx, ecx				; The Shift Right operation
	mov esi, OFFSET Nodes_X
	mov al, [esi+ebx-1]			; to make accomodation for
	mov [esi+ebx], al
	mov esi, OFFSET Nodes_Y		; new node. All of the nodes are
	mov al, [esi+ebx-1]
	mov [esi+ebx], al			; shifted right and
	Loop ShiftRight
	mov esi, OFFSET Nodes_X		; the new node is
	mov al, x_apple
	mov [esi], al				; located to its place.
	mov esi, OFFSET Nodes_Y
	mov al, y_apple
	mov [esi], al	
NotEaten:
	call Configure				; The nodes of the snake are
	call PrintNodes				; updated on each move and printed
Finish:							; on the screen.
	ret
AddNodes ENDP
Configure PROC USES eax ebx ecx esi
	mov esi, OFFSET Nodes_X		; The configuration of the snake is
	mov al, [esi]
	mov x_tail, al				; done by this procedure.
	mov esi, OFFSET Nodes_Y
	mov al, [esi]				; The old tail is saved to be erasen.
	mov y_tail, al
	mov ebx, 1
	mov ecx, NumOfNodes
	inc ecx
ShiftLeft:
	mov esi, OFFSET Nodes_X		; This Shift Left operation is applied
	mov al, [esi+ebx]
	mov [esi+ebx-1], al			; to renew the locations of
	mov esi, OFFSET Nodes_Y
	mov al, [esi+ebx]			; each nodes.
	mov [esi+ebx-1], al
	mov al, [esi]				; So, the snake is always
	inc ebx
	Loop ShiftLeft				; in motion.
	mov ebx, NumOfNodes
	mov esi, OFFSET Nodes_X
	mov al, x_head
	mov [esi+ebx], al
	mov al, [esi]
	mov esi, OFFSET Nodes_Y
	mov al, y_head
	mov [esi+ebx], al
ret
Configure ENDP
PrintNodes PROC USES eax ebx ecx edx esi
	mov dl, x_tail				; The coordinates of the
	mov dh, y_tail
	call Gotoxy					; nodes and the head are
	mov al, ' '
	call WriteChar				; taken from the arrays and
	mov ecx, NumOfNodes
	inc ecx						; they are printed on the screen.
Print:
	mov ebx, ecx
	mov esi, OFFSET Nodes_X
	mov al, [esi+ebx-1]			; Printing loop.
	mov dl, al
	mov esi, OFFSET Nodes_Y
	mov al, [esi+ebx-1]			; The addresses of the arrays are
	mov dh, al
	call Gotoxy					; enough to reach each node and head.
	mov edx, NumOfNodes			; The head is printed as its own character
	inc edx
	cmp ecx, edx				; and the nodes are printed as itselves.
	jne PrintNode
	mov al, head
	jmp Printt
PrintNode:
	mov al, node
Printt:
	call WriteChar
	Loop Print
ret
PrintNodes ENDP
CrashSnake PROC USES eax ebx ecx edx esi
	mov ecx, NumOfNodes			; This procedure controls if the
	cmp ecx, 3
	jle Finish					; snake eats any of its nodes.
	inc ecx
Crash:							; If it eats any of its nodes or tail
	mov ebx, ecx	
	mov esi, OFFSET Nodes_X		; it crashes and the game ends.
	mov al, [esi+ebx-2]
	mov dl, al
	mov esi, OFFSET Nodes_Y		; The coordinates of the head are
	mov al, [esi+ebx-2]
	mov dh, al					; compared with the nodes and tail
	mov al, x_head
	mov ah, y_head				; which are held into the arrays that
	cmp dx, ax
	je Lengthh					; are named Nodes_X and Nodes_Y.
	jmp Endd
Lengthh:						; The snake cannot eat its head so,
	mov edx, NumOfNodes
	inc edx						; after 3 nodes this control is started.
	cmp ecx, edx
	je Endd
EndOfGame:
	mov dl, x_head				; If the snake eats its nodes or tail,
	mov dh, y_head
	call Gotoxy					; the game ends because of the crash.
	mov al, head
	call WriteChar
	mov dl, 33
	mov dh, 13					; The game ending signs are configured
	call Gotoxy
	mov edx, OFFSET String3		; and printed on the screen.
	call WriteString
	pushad
	call readchar
	cmp al,'y'
	je begin
	popad
	exit
Endd:
	Loop Crash
Finish:
	ret
CrashSnake ENDP
GameScore PROC USES eax			; This procedure controls
	mov eax, score
	add eax, 1					; the score of the game.
	mov score, eax				; Each eaten apple is "1" point.
	ret							; It is performed accoding to the
GameScore ENDP					; number of the apples eaten.

GameSpeed PROC USES eax ebx edx
	mov edx, 0
	mov eax, score
	mov ebx, 10	
	div ebx
	cmp edx, 1
	jne Finish
	mov ax, speed
	mov bx, 10
	sub ax, bx					; After each 10 apple, the speed of the game increases.
	mov speed, ax
	mov eax,score			
	add eax,1
	mov score,eax 
Finish:
	ret
GameSpeed ENDP
	exit
END main
