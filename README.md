### 记录下火影·新世代游戏中的action文件分析过程.

首先该游戏是用Unity5.6.x搞的，所以是用的mono2.0，mono自己实现了一套.net framework 2.0和.net framework 3.5.

下面来瞅瞅action文件的二进制内容
![二进制内容](/图片/1.png)

这里可以看到该文件是基于c#的BinaryFormatter来序列化的，那么就基于这个方向去找BinaryFormatter是怎样序列化的就可以了。
由于mono实现的BinaryFormatter和微软自带的BinaryFormatter可能会有差别，但是在网上找到了个微软的BinaryFormatter文档说明，那就直接拿它试试水。

```

Binary Serialization Format
    RecordTypeEnum: SerializedStreamHeader (0x00)
    TopId: 1 (0x00000001)
    HeaderId: -1 (0xFFFFFFFF)(这里类型是Int32，所以结果是-1)
    MajorVersion: 1 (0x00000001)
    MinorVersion: 0 (0x00000000)
BinaryLibrary:
    RecordTypeEnum: BinaryLibrary (0x0C)
        MessageEnum: 0x00000002
            NoArgs: (.................................0) 2^0
            ArgsInline: (............................1.) 2^1
            ArgsIsArray: (..........................0..) 2^2
            ArgsInArray: (.........................0...) 2^3
            NoContext: (..........................0....) ...
            ContextInline: (.....................0.....) ...
            ContextInArray: (...................0......) ...
            MethodSignatureInArray: (..........0.......) ...
            PropertyInArray: (................0........) ...
            NoReturnValue: (.................0.........) ...
            ReturnValueVoid: (..............0..........) ...
            ReturnValueInline: (...........0...........) ...
            ReturnValueInArray: (.........0............) ...
            ExceptionInArray: (..........0.............) ...
            Reserved: (000000000000000000..............) ...
        TypeName:
            PrimitiveTypeEnum: String (0x0F)
            Data: Assembly-CSharp
    RecordTypeEnum: ClassWithMembersAndTypes (0x05)
        ClassInfo:
            ObjectId: 1 (0x00000001)
            TypeName: Naruto.Config.actions (0x15)
            MemberCount: 1 (0x00000001)
            MemberNames: aktions (0x07)
        MemberTypeInfo:
            BinaryTypeEnums: 4 (0x04) CustomClass
            //因为上面已经判断了是个自定义类型，所以后面就是ClassTypeInfo数据
            AdditionalInfos: ClassTypeInfo
                TypeName: Naruto.Config.action[] (0x16)
                LibraryId: 2 (0x00000002)
        LibraryId: 2 (0x00000002)
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 3 (0x00000003)
    RecordTypeEnum: BinaryArray (0x07)
        ObjectId: 3 (0x00000003)
        BinaryArrayTypeEnum: Single (0x0) //一维数组
        Rank: 1 (0x00000001) 数量的维度
        Lengths: 73 (0x00000049) 指定数组中每个维的长度（猜测，如果是二维的，这里后面应该还会跟一个4字节的数据）
        LowerBounds: BinaryArrayTypeEnum为 SingleOffset, JaggedOffset, RectangularOffset 时才有该字段
        TypeEnum: 4 (0x4) (根据BinaryTypeEnum表查找到是CustomClass类型)
        //因为TypeEnum是Class类型，所以Additional数据类型为ClassTypeInfo
        AdditionalTypeInfo: (ClassTypeInfo)
            TypeName: Naruto.Config.action (0x14)
            LibraryId: 2 (0x00000002)
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 4 (0x00000004)
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 5 (0x00000005)
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 6 (0x00000006)
                ...
                ...
                ...
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 74 (0x0000004A) 
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 75 (0x0000004B) 
    RecordTypeEnum: MemberReference (0x09)
        IdRef: 76 (0x0000004C) 
    RecordTypeEnum: ClassWithMembersAndTypes (0x05)
        ClassInfo:
            ObjectId: 4 (0x00000004)
            Name: Naruto.Config.action (0x14)
            MemberCount: 4 (0x00000004)
            MemberNames:
                <id>k__BackingField (0x13)
                <name>k__BackingField (0x15)
                <hide>k__BackingField (0x15)
                <layers>k__BackingField (0x17)
        MemberTypeInfo:
            BinaryTypeEnums: (这里的长度等同于MemberCount的数量)
                0 (0x0) (Primitive类型)
                1 (0x1) (String类型)
                0 (0x0) (Primitive类型)
                4 (0x4) (CustomClass类型)
            //根据上面的类型,这里开始识别字段具体类型
            AdditionalInfos:
                1、8 (0x8) (Int32类型)
                2、因为这里是个String类型，所以忽略了
                3、1 (0x1) (Boolean类型)
                4、Naruto.Config.layer[] (0x15) (CustomClass类型)
        LibraryId: 2 (0x00000002)
    RecordTypeEnum: SystemClassWithMembers (0x02)
        ClassInfo:
            ObjectId: 0 (0x00000000)
            Name: 

```

从上面的分析已经可以得出整个action文件的基本结构类型图了.

```
namespace Naruto.Config
{
    [Serializable]
    public class actions
    {
        public action[] aktions;
    }

    [Serializable]
    public class action
    {
        public int id { get; set; }

        public string name { get; set; }

        public bool hide { get; set; }

        public layer[] layers { get; set; }
    }

    [Serializable]
    public class layer
    {
        public int id { get; set; }

        public string name { get; set; }

        public bool visible { get; set; }

        public bool @lock { get; set; }

        public string type { get; set; }

        public frame[] frames { get; set; }
    }

    [Serializable]
    public class frame
    {
        public int index { get; set; }

        public int length { get; set; }

        public string @event { get; set; }

        public element[] elements { get; set; }

        public sound[] sounds { get; set; }

        public pointElement[] pointElements { get; set; }
    }

    [Serializable]
    public class element
    {
        public int x { get; set; }

        public int y { get; set; }

        public int assetId { get; set; }

        public string filename { get; set; }

        public float scaleX { get; set; }

        public float scaleY { get; set; }

        public float rotation { get; set; }

        public float alpha { get; set; }
    }

    [Serializable]
    public class sound
    {
        public int id { get; set; }

        public int volume { get; set; }

        public string name { get; set; }
    }

    [Serializable]
    public class pointElement
    {
        public int x { get; set; }

        public int y { get; set; }

        public string name { get; set; }
    }
}
```

拿到该结构和action.bin文件，直接反序列化一下就可以拿到action的数据内容了.

![结果](/图片/2.png)
