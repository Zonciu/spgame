How to use RecvPacket class:

```C++
CRecvPacket pRecvPacket = new CRecvPacket(recvBuff, dwBytes);
long pRecvType = pRecvPacket->GetType();
size_t pRecvSize = pRecvPacket->GetSize();

if ( pRecvPacket->IsValid() )
  switch (pRecvType)
  {
    case 0x2707:
      char loginUsername[13] = *(long *)(pRecvPacket->GetPPtr()+60);
      char loginPassword[13] = *(long *)(pRecvPacket->GetPPtr()+73);
      break;
    case 0x2911:
      long pRecv_preChar = (long)*(unsigned char *)(pRecvPacket->GetPPtr()+36);
      		unsigned char *myData = (unsigned char*)calloc(0x20-0x14, 0x1);
    			memcpy(myData+0x14-0x14, new short(1), 2); //must be 01 00 00 00
    			memcpy(myData+0x18-0x14, &pRecv_preChar, 4);//character 10,20,30...etc
    			CPacket *myPacket = new CPacket(0x2912, myData, 0x20-0x14);
    			free(myData);
    			myPacket->Encrypt();
    			myPacket->Output(myData, myPacket->GetSize());
    			    WriteComm1((char*)myData, myPacket->GetSize(), dwTimeout);
      break;
    default:
  }
```