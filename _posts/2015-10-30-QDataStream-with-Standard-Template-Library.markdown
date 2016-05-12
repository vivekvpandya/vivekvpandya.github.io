---
layout: post
title:  "QDataStream with Standard Template Library"
date:   2015-10-30 10:18:00
categories: Qt Serialization
---
Serialization is really helpful when storing data to files or sending it over the network. When writing any network application if control messages are sent as a plain text it becomes very cumbersome to create object from it. So it may be useful to send the whole object as a binary data and reconstruct it at other end. In Qt to achieve this goal QDataStream can be used.

It is always advisable to use Qt template containers like QVector, QList etc when programing in Qt but some times you may want to re-use some C++ code with less modification and that code does not uses Qt containers instead it uses Standard Template Library. QDataStream class do not provide support for STL out of the box. 

QDataStream provides serialization of binary data to QIODevice. QDataStream provides implementation of in stream ```<<``` and out stream ``` >> ``` operator for primitive data types like int, float , bool etc. It also provides implementation of ```>>``` and ```<<``` operator for Qt template library.  

QDataStream internally operates on QIODevice. It can be used with any type of QIODevice like sockets, buffers etc. While working with STL in your classes, we can code how to serialize STL containers by overloading ```<<``` and ```>>``` operators for custom classes. In fact to work with QDataStream object the custom class needs to overload ```<<``` and ```>>``` operator. The idea to make STL work is quite intuitive. As STL containers have dynamic size (resolved at runtime) so, before serializing the original data, the size of data is serialized  and it is used when deserializing the data. While doing deserialization on binary data the size of containers will be used to identify the boundary of the containers. This will be clear in the example.

Here is an example code for Message class. Full code can be downloaded  <a href="{{ site.root }}/downloads/DataStreamTest.zip"> here </a>

{% highlight c++ %}
QDataStream & operator <<(QDataStream & stream, const Message &message){
    stream << message.getMessageType(); // writing enum value
    std::vector<QString> dataStrings = message.getDataStrings();
    int dataStringsSize = dataStrings.size();
    stream << dataStringsSize; // writing dataString vector size first
    std::vector<Peer> peerObjects = message.getPeerVector();
    int peerVectorSize = peerObjects.size();
    stream << peerVectorSize; // writing peer vector size
    std::vector<Room> roomVector = message.getRoomVector();
    int roomVectorSize = roomVector.size();
    stream << roomVectorSize; // writing room vector size
    // writing the actual dataStrings
    for( QString obj : dataStrings ){  
        stream << obj;
    }
    // writing the actual peer objects
    for(Peer peer :  peerObjects){
        stream<<peer;
    }
    // writing the actual room objects
    for(Room room: roomVector){
        stream << room;
    }
    return stream;
}

QDataStream &  operator >>(QDataStream & stream, Message &message){
    MessageType mtype;
    QString stringObj;
    Peer peerObj;
    Room roomObj;
    int dataStringsSize;
    int peerVectorSize;
    int roomVectorSize;
    stream >> mtype; // reading enum object
    stream >> dataStringsSize; // reading dataString vector size
    stream >> peerVectorSize; // reading Peer vector size
    stream >> roomVectorSize; // reading Room vector size
    message.setMessageType(mtype);
    for(int i=0;i<dataStringsSize;i++){
        stream>>stringObj; // reading actual dataString
        message.insertDataString(stringObj);
    }
    for(int i=0;i<peerVectorSize; i++){
        stream>>peerObj; // reading actual peer objects
        message.insertPeerObj(peerObj);
    }
    for(int i=0;i<roomVectorSize; i++){
        stream >> roomObj; // reading actual room objects
        message.insertRoomObj(roomObj);
    }
    return stream;
}
{% endhighlight %}

This code shows how to overload operator ```<<``` and ```>>``` for the Message class which contains one enum data, and some vectors. Note that before writing actual data from vector it is writing the size of that vector and based on that while reading the data the size will help to drive the loop. Also here order of reading should be same as of writing. Every custom classes that needs to be serialized should overload ```<<``` and ```>>``` operator. So in above code Peer and Room class also overloads the ```<<``` and ```>>``` operator. See the full code for more details.

Here is the code for writing the serialized data to a simple file.
{% highlight c++ %}
    QDataStream io;
    QFile arch;
    arch.setFileName("/Users/Mr.Pandya/Desktop/object"); 
    arch.open(QIODevice::WriteOnly);
    io.setDevice(&arch);
    Message message =  Message();
    message.setMessageType(MessageType::GetRoomDetails);
    message.insertDataString("Vivek");
    message.insertDataString("Pandya");
    message.insertDataString("Qt");
    Peer peer  = Peer();
    peer.setNickName("Batman");
    peer.setPeerAddress(QHostAddress::AnyIPv4);
    message.insertPeerObj(peer);
    Room room = Room();
    room.setPort(12000);
    room.setRoomName("Qt-Developers");
    room.setRoomDesc("Qt developer's discussion room");
    room.addNickName(peer);
    message.insertRoomObj(room);
    io << message; // write object on data stream
    arch.flush();
    arch.close();
{% endhighlight %}

   
I have used QDataStream to send objects over network , here is the code <a href="https://github.com/vivekvpandya/RoomManager/blob/master/mainwindow.cpp" >RoomManager</a>

The output in file will be binary data. Some text editors may be able to decode String objects. 
Here is the code to read Message object from the file:
{% highlight c++ %}
 // Read from the file
    QDataStream io;
    QFile arch;
    arch.setFileName("/Users/Mr.Pandya/Desktop/object"); 
    arch.open(QIODevice::ReadOnly);
    io.setDevice(&arch);
    Message readMessage = Message();
    io >> readMessage; // read from data stream and store in readMessage object
    qDebug() << (int)readMessage.getMessageType();
    for(QString dataString: readMessage.getDataStrings())
        qDebug() << dataString << "\n";
    for(Peer peer : readMessage.getPeerVector())
        qDebug() << peer.getpeerAddress() << "\n" << peer.getNickName() << "\n";
{% endhighlight %}


This may be a novice approach but still I hope this would help. Any suggestions, improvements are always welcomed.




