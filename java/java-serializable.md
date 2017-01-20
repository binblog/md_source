```
public ObjectOutputStream(OutputStream out) ... {
	writeStreamHeader();	// STREAM_MAGIC:aced,STREAM_VERSION:5
}


public final void writeObject(Object obj)... {
	...
	writeObject0(obj, false);
}


private void writeObject0(Object obj, boolean unshared) {
	...
	ObjectStreamClass desc;
	desc = ObjectStreamClass.lookup(cl, true);

	if (obj instanceof String) {
	                writeString((String) obj, unshared);
	            } else if (cl.isArray()) {
	                writeArray(obj, desc, unshared);
	            } else if (obj instanceof Enum) {
	                writeEnum((Enum<?>) obj, desc, unshared);
	            } else if (obj instanceof Serializable) {
	                writeOrdinaryObject(obj, desc, unshared);
	            } else {
					...
				}
}

private void writeOrdinaryObject(Object obj,ObjectStreamClass desc,boolean unshared) {
	bout.writeByte(TC_OBJECT);	// TC_OBJECT:0x73			
	writeClassDesc(desc, false);				
	
	writeSerialData(obj, desc);
}

private void writeClassDesc(ObjectStreamClass desc, boolean unshared) ... {
    int handle;
    if (desc == null) {
        writeNull();
    } else if (!unshared && (handle = handles.lookup(desc)) != -1) {
        writeHandle(handle);
    } else if (desc.isProxy()) {
        writeProxyDesc(desc, unshared);
    } else {
        writeNonProxyDesc(desc, unshared);
    }
}

private void writeNonProxyDesc(ObjectStreamClass desc, boolean unshared) {
	bout.writeByte(TC_CLASSDESC);	// TC_CLASSDESC:0x72
	...
	writeClassDescriptor(desc);
	
	bout.writeByte(TC_ENDBLOCKDATA); // 78
	
	writeClassDesc(desc.getSuperDesc(), false);	// 父类 70

}

protected void writeClassDescriptor(ObjectStreamClass desc)
    throws IOException
{
    desc.writeNonProxy(this);


}
```

```
void writeNonProxy(ObjectOutputStream out) {
	out.writeUTF(name);	// serialize.User 000E73657269616C697A652E55736572
	out.writeLong(getSerialVersionUID());	// 8C6E58F2935F1317
	
	out.writeByte(flags);	// 	02
	out.writeShort(fields.length);	// 00 01
	
	for (int i = 0; i < fields.length; i++) {
            ObjectStreamField f = fields[i];
            out.writeByte(f.getTypeCode());	// L:4C
            out.writeUTF(f.getName());	// 00046E616D65
            if (!f.isPrimitive()) {
                out.writeTypeString(f.getTypeString());	// 7400124C6A6176612F6C616E672F537472696E673B
            }
        }

}
```

out.writeTypeString {
	        bout.writeByte(TC_STRING);
            bout.writeUTF(str, utflen);
}

```
private void writeSerialData(Object obj, ObjectStreamClass desc) {
	if (slotDesc.hasWriteObjectMethod()) {
		slotDesc.invokeWriteObject(obj, this);
	} else {
		defaultWriteFields(obj, slotDesc);
    }	
}

private void defaultWriteFields(Object obj, ObjectStreamClass desc) {
	desc.getPrimFieldValues(obj, primVals);	// 原始数据，如int，double
	bout.write(primVals, 0, primDataSize, false);

	desc.getObjFieldValues(obj, objVals);	
writeObject0(objVals[i],                            fields[numPrimFields + i].isUnshared());	// 0003 62 69 6E
}
```


ObjectStreamClass用于存储一些Class解析结果，如是否继承了Serializable， 是否为enum，字段信息等。