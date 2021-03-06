Navigation<br />
<a href="https://github.com/umehkg/spgame/blob/master/comments/recvPacket.md#type-i--login-server-tcp">1. Login Server (TCP)</a> <br />
<a href="https://github.com/umehkg/spgame/blob/master/comments/recvPacket.md#type-ii-game-server-udp">2. Game Server (UDP)<br />
<a href="https://github.com/umehkg/spgame/blob/master/comments/recvPacket.md#type-iii-lobby-server-tcp"> 3. Lobby Server (TCP) </a><br />

NOTE: CClientTCPSocket::GetSocket should be CGenericMesssage::GetType, due to problems in IAT.
CGenericMesssage::GetType(void) returns unsigned long packetType
```C++
unsigned long packetType = *(LPDWORD)(packet+0x4);
```
Type I : Login Server (TCP)
------
packetType = 0x2807 / 0x29XX **TCP Packet** <br />
```asm
;iosocketdll.dll
;CClientTCPSocket::ReceiveProcess
___:015B2327                 mov     eax, [esi]
___:015B2329                 push    edi
___:015B232A                 mov     ecx, esi
___:015B232C                 call    dword ptr [eax+14h]

; ecx: 0x007F3C38
; eax: 0x0076D6C0
; [eax+14h]: 0x006FAC00 (Type I)
; edi: TCP packet payload received by client

; N.B. [eax+30h]: 0x006F77D0 (Type III)
```
Recv Ref: CClientTCPSocket::ReceiveProcess-> call <a href="https://github.com/umehkg/spgame/blob/master/src/sp/2FAC00_RecvPacket.cpp">sub_6FAC00</a> <br />
Complete listing of Type I functions (10 of them): Confirmed TCP <br />
$: complete, +: working, -: unsolved
- 0x2807 sub_6F9910 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x2807.txt">+</a> Login Response
- 0x2908 sub_6F8E80 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x2908.txt">+</a> Channel Info (one pkt per channel)
- 0x2912 sub_6F90B0 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x2912.txt">$</a> SetActiveCharacter
- 0x2916 sub_6F9FC0 - related with 2 cards. fusion, skillup?
- 0x2918 sub_6F8B50 <a href="https://github.com/umehkg/spgame/edit/master/comments/packetType/0x2918.txt">$</a> Set Total Population Bar Below Channel Select
- 0x2919 sub_6F8AF0 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x2919.txt">+</a> size 0x18
- 0x2922 sub_6F8BD0 - size 0x2C "Guild Mark Lock..
- 0x2923 sub_6FAAF0 - > call sub_6FA650 card/white card related
- 0x2924 sub_6FA9F0 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x2924.txt">+</a> size 0x18 TCPSendLoop(size 0x1C)
- 0x2925 sub_6FAAD0 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x2925.txt">$</a> Kick Client? (bring to login screen). Displays korean msg. (size 0x14) > jmp sub_4035D0

Type II: Game Server (UDP)
--------
packetType = 0x1100~0x1140　**UDP Packet** Why do they share the same function? It's a mystery.<br />
Ref: sub_70A4D0<br />
```asm
.text:0070A527                 call    ?ReceiveLoop@CUDPSocket@@QAE_NIAAVCGenericRcvMsg@@@Z ; CUDPSocket::ReceiveLoop(uint,CGenericRcvMsg &)
.text:0070A52C                 test    al, al
.text:0070A52E                 jz      loc_70A8E3
.text:0070A534                 mov     [ebp+var_4031], 1
.text:0070A53B                 xor     ebx, ebx
.text:0070A53D                 mov     [ebp+var_4], ebx
.text:0070A540                 lea     ecx, [ebp+var_4030]
.text:0070A546                 nop
.text:0070A547                 call    ?GetSocket@CClientTCPSocket@@QAEIXZ ; CClientTCPSocket::GetSocket(void)
.text:0070A54C                 add     eax, 0FFFFEF00h
.text:0070A551                 cmp     eax, 40h        ; switch 65 cases
```
Complete listing of Type II ( packetType = 0x1100 + dec2hex(caseNumber) ) Confirmed UDP
- 0x1100 case 0: sub_6FAE00
- 0x1101 case 1: sub_6FAFB0 (no body) Enter Channel Response?
- 0x1102 case 2: sub_702F60
- 0x1103 case 3: sub_6FC660
- 0x1104 case 4: sub_6FC820
- 0x1105 case 5: sub_6FCA70
- 0x1106 case 6: sub_6FCD00
- 0x1107 case 7: sub_6FD220
- 0x1108 case 8: sub_6FD5C0
- 0x1109 case 9: sub_7089B0
- 0x1110 case 16: sub_705A00
- 0x1112 case 18: sub_6FECB0
- 0x1113 case 19:
- 0x1114 case 20:
- 0x1115 case 21:
- 0x1118 case 24:
- 0x1119 case 25:
- 0x1120 case 32: sub_6FFEB0 GameState == 4 ? 
- 0x1121 **case 33**: sub_700430 Login response
- 0x1123 case 35: sub_6FB590 MyInfo->0x2E01~0x2E08, checks MyInfo->0xCC0 == 0x244 ?
- 0x1124 case 36: sub_6FF5F0 GameState == 3 || 4 ?
- case 37: sub_7068C0 "CUDPQuestNPCCollisionMsg Rcv Start"
- case 38: sub_701D00 GameState == 4 ? "CUDPNpcControlMsg Rcv Start"
- case 39: sub_701E80 GameState == 4 ?
- case 40: sub_6FB050 some game message in korean
- case 41: sub_702090 GameState == 4 ?
- 0x1130 case 48: sub_702230 GameState == 4 ?
- 0x1131 case 49: sub_70A1B0 GameState == 4 ?
- 0x1132 case 50: sub_6FB520 ds:7EF97C ???
- 0x1133 case 51:
- 0x1134 case 52:
- 0x1135 case 53:
- 0x1136 case 54:
- 0x1137 case 55:
- 0x1138 case 56:
- 0x1139 case 57:
- 0x1140 case 64: sub_706150 GameState == 4 ?

Type III: Lobby Server (TCP)
-----------
packetType: 0x4302~0x4490<br />

Init Ref: sub_484990 <br />
```asm
.text:004849AD                 mov     eax, [esi+2FE4h]
.text:004849B3                 mov     ecx, [esi+2FE8h]
.text:004849B9                 lea     eax, [eax+eax*4]
.text:004849BC                 lea     eax, [ecx+eax*8]
.text:004849BF                 mov     edx, [esi+eax*4+0AC0h]
.text:004849C6                 mov     ecx, ds:wParam
.text:004849CC                 push    edx
.text:004849CD                 shl     eax, 4
.text:004849D0                 lea     eax, [eax+esi+0B4h]
.text:004849D7                 push    eax
.text:004849D8                 push    40Bh
.text:004849DD                 push    ecx
.text:004849DE                 mov     ecx, offset off_800518
.text:004849E3                 nop
.text:004849E4                 call    ?InitClientTCP@CClientTCPSocket@@QAE_NPAUHWND__@@IPADH@Z ; CClientTCPSocket::InitClientTCP(HWND pWnd*,uint wMsg,char * ip_addr, int port)
```
```C++
  long channelIdx = *(long *)(0x7F0BD8+0x2FE4) * 40 + *(long *)(0x7F0BD8+0x2FE8);
  char ip_addr[15];
  long port = *(long *)(0x7F0BD8+0xAC0+channelIdx*4);
  
  memcpy(&ip_addr[0], (char *)(0x7F0BD8+0xB4+channelIdx*16), 16);
  (CClientTCPSocket *)(LPVOID)0x800518->InitClientTCP((HWND *)(LPVOID)0x800518, 1035, ip_addr, port);
```
Recv Ref: ds:800518->+0x14=ds:76D6F0->**sub_6F77D0**<br />
Note: The caller of this function is CClientTCPSocket::ReceiveProcess<br />
```asm
.text:006F7808                 call    ?GetSocket@CClientTCPSocket@@QAEIXZ ; CClientTCPSocket::GetSocket(void)
.text:006F780D                 add     eax, 0FFFFBCFEh
.text:006F7812                 cmp     eax, 18Eh       ; switch 399 cases
```
*Complete* listing of Type III: **TCP Packet** <br />
- 0x4302: case 0: sub_6DDE30 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x4302.txt">+</a> MyInfo
- 0x4303: case 1: sub_6D8B50 <a href="https://github.com/umehkg/spgame/edit/master/comments/packetType/0x4303.txt">+</a> LobbyPlayerList
- 0x4304: case 2: sub_6DC770 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x4304.txt">+</a> LobbyInfo (rooms)
- <s>0x4305: case 3: sub_6D8C50 Nothing</s>
- 0x4306: case 4: sub_6D8BC0 PlayerList??
- 0x4308: case 6: sub_6DEAA0 "CreateRoom - End"
- 0x4310: case 14: sub_6D8D40 MessageWnd/UserInfoWnd //cmp playerUsername
- 0x4312: case 16: sub_6DCC50 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x4312.txt">+</a> ; RoomInfoWnd <"Over Room Array Number", "CGameRoomInfoResultMsg - Result Type:%d">
- 0x4314: case 18: sub_6DEC00 "JoinRoom - End" (RoomInfo)
- 0x4317: case 21: sub_6F3F60 EnterRoom (PlayerInfo)
- 0x4319: case 23: sub_6DF0A0 //ExitRoom (GameState == 3)
- 0x4321: case 31: sub_6DF700
- 0x4323: case 33: sub_6DF900 //ExitRoom (GameState == 4)
- 0x4324: case 34: sub_6F6F30
- 0x4326: case 36: sub_6D90E0
- 0x4328: case 38: sub_6D9190
- 0x4330: case 46: sub_6D91D0
- 0x4332: case 48: sub_6D9430 
- 0x4334: case 50: sub_6EA920
- 0x4336: case 52: sub_6EAC10 
- 0x4337: case 53: sub_6E0570
- 0x4338: case 54: sub_6EC0F0
- 0x4340: case 62: sub_6ED420 GameState == 4 ? confirm, playerName
- 0x4341: case 63: sub_6E1980
- 0x4343: case 65: sub_6E0A50 + CardFusion StartFusion Response "Fusion Result : %d">
- 0x4345: case 67: sub_6E0E30
- 0x4346: case 68: sub_6E11A0
- 0x4347: case 69: sub_6E1A90
- 0x4349: case 71: sub_6E1F50
- 0x4352: case 80: sub_6DA2F0
- 0x4354: case 82: sub_6DA490
- 0x4356: case 84: sub_6DA580
- 0x4359: case 87: sub_6EDFA0
- 0x4361: case 95: sub_6EE9C0
- 0x4362: case 96: sub_6EF1A0  ; <"CGameQuestAllDataMsg Rcv Start", "CGameQuestAllDataMsg Rcv End"
- 0x4364: case 98: sub_6DCE70 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x4364.txt">$</a> EnterCardShopResponse
- 0x4366: case 100: sub_6EA660
- 0x4368: case 102: sub_6DCEB0  <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x4368.txt">$</a> CardShopBuyResult
- 0x4369: case 103: sub_6DD1F0
- 0x4371: case 111: sub_6D9DA0 
- 0x4372: case 112: sub_6E1BC0 ; <"void CGameUserShopResultMsg::Process-1", ".\CClientGameMsg.cpp", "ASSERT Fail,return
- 0x4373: case 113: sub_6D9A50 ; <"CGameUserShopInfoMsg::Process -0", "CGameUserShopInfoMsg state:%d, id:%s, shop_name:%s", 
- 0x4376: case 116: sub_6E2460
- 0x4377: case 117: sub_6DA6A0
- 0x4378: case 118: sub_6DD260
- 0x4379: case 119: sub_6E0780
- 0x4380: case 126: sub_6F7320
- 0x4381: case 127: sub_6EFAD0
- 0x4382: case 128: sub_6F0C30  ; <"CGameEnjoyAllDataMsg Rcv Start", "CGameEnjoyAllDataMsg Rcv
- 0x4384: case 130: sub_6DA7B0
- 0x4385: case 131: sub_6E25B0  ; <"Software\IOSPK\EVENT", "SPCARD">
- 0x4387: case 133: sub_6DAAE0
- 0x4389: case 135: sub_6DABF0 Siege Wnd <-- View Guild Btn
- 0x4390: case 142: sub_6DD490
- 0x4391: case 143: sub_6E2BC0
- 0x4393: case 145: sub_6DD570
- 0x4395: case 147: sub_6F1F20; <"CGameBigBattleNpcAllMsg Rcv End", "CGameBigBattleNpcAllMsg Rcv Start 3", "CGameBigBattleNpcAllMsg 
- 0x4397: case 149: sub_6F28F0
- 0x4399: case 151: sub_6DAD80 GMWnd AdminPage OK
- 0x4403: case 257: sub_6DAF10
- 0x4404: case 258: sub_6E2D50
- 0x4405: case 259: sub_6DAFB0
- 0x4407: case 261: sub_6DD5C0
- 0x4408: case 262: sub_6F3A90
- 0x4409: case 263: sub_6DAFD0
- 0x4410: case 270: sub_6DAFF0
- 0x4412: case 272: sub_6DB060
- 0x4414: case 274: sub_6DB140
- 0x4416: case 276: sub_6DB1F0
- 0x4418: case 278: sub_6E2DB0
- 0x4419: case 279: sub_6DB2C0
- 0x4420: case 286: sub_6E2FC0
- 0x4422: case 288: sub_6F3C80
- 0x4423: case 289: sub_6E30B0
- 0x4425: case 291: sub_6DB4C0
- 0x4426: case 292: sub_6DB5A0 ; <"CGameMemoArrivalResult::Process TYPE ERROR (%d)">
- 0x4427: case 293: sub_6DD640 ; <"MAGIC", "Software\IOSPK\EVENT\FREE811\%s">
- 0x4429: case 295: sub_6E3600
- 0x4431: case 303: sub_6F3E70 ; <"CGamePetKilledResult PET ARRAY OVER(%d)!!", "C
- 0x4433: case 305: sub_6DDD40
- 0x4436: case 308: sub_6E37C0
- 0x4439: case 311: sub_6E39C0
- 0x4441: case 319: sub_6DB7F0
- 0x4442: case 320: sub_6E3A80      ; <".                  .", "Event Error : %d : %d : %d", "MyInfo is NULL %d", "GameState is error
- 0x4443: <s>case 321: nullsub_5 Nothing</s>
- 0x4445: case 323: sub_6DB990
- 0x4447: case 325: sub_6E7E90
- 0x4448: case 326: sub_6DBCA0
- 0x4450: case 334: sub_6DBE40 ; <"[ %s ]                   .">
- 0x4452: case 336: sub_6E7FC0
- 0x4453: case 337: sub_6DBEF0
- 0x4455: case 339: sub_6E8050
- 0x4457: case 341: sub_6DBF80
- 0x4458: case 342: sub_6E84C0
- 0x4460: case 350: sub_6DC050
- 0x4461: case 351: sub_6E8730 ; <"Bonus Visit State Error.", ".\CClientGameMsg.cpp",
- 0x4463: case 353: sub_6E88C0
- 0x4465: case 355: sub_6E8990      ; <"/                              .", "/                          .", "'%s
- 0x4466: case 356: sub_6DC140      ; <"[%s]">
- 0x4468: case 358: sub_6DC300 $ CardSlotExtend Apply Response (?
- 0x4470: case 366: sub_6DC3D0
- 0x4471: case 367: sub_6E90B0
- 0x4472: case 368: sub_6DC420
- 0x4473: case 369: sub_6E9800
- 0x4475: case 371: sub_6EA040
- 0x4476: case 372: sub_6EA1E0  ; <"CGameBoxNpcCreateMsg::Process ERROR:%d", "CGameBoxNpcCreateMsg::Process NewNPCNode[%d] != HIDE", "CGameBoxNpcCreateMsg::Process NewNPCNode[%d] != BOX_TYP>
- 0x4477: case 373: sub_6EA470
- 0x4479: case 375: sub_6E9F00
- 0x4481: case 383: sub_6DCA20
- 0x4483: case 385: sub_6DC570
- 0x4485: case 387: sub_6DC5D0
- 0x4487: case 389: sub_6DC6D0 + ; <"Guild Mark Lock Fail!">
- 0x4488: case 390: sub_6DC700
- 0x4490: case 398: sub_6EA540 <a href="https://github.com/umehkg/spgame/blob/master/comments/packetType/0x4490.txt">+</a>
