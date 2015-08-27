# Metin2 Server & Client: Transform / Enchant Costume

Diese Anleitung zeigt, wie man zwei neue Items implementiert:
- **Transform Costume** (VNum: 70063): Verändert Boni und kann ggf. zusätzliche hinzufügen (max. 3).
- **Enchant Costume** (VNum: 70064): Erneuert bestehende Boni.

📅 Veröffentlichung: [27.08.2015, 15:52](https://www.elitepvpers.com/forum/metin2-pserver-guides-strategies/3845493-how-kost-m-verwandeln-verzaubern-transform-enchant-costume.html)

---

## 🔧 Serverseitige Änderungen

### 1. `char_item.cpp`
**Suche:**
```cpp
if (ITEM_COSTUME == item2->GetType())
```
**Ersetzen mit:**
```cpp
if (ITEM_COSTUME == item2->GetType() && item->GetVnum() != 70063 && item->GetVnum() != 70064)
```

**Suche:**
```cpp
if (item2->GetAttributeSetIndex() == -1)
```

**Darunter einfügen:**
```cpp
// Transform costume
if (item->GetVnum() == 70063) {
    if (item2->GetType() != ITEM_COSTUME) {
        ChatPacket(CHAT_TYPE_INFO, LC_TEXT("ITEM_ISNT_COSTUME"));
        return false;
    }
    if (item2->GetAttributeCount() == 0) {
        ChatPacket(CHAT_TYPE_INFO, LC_TEXT("ITEM_HASNT_ATTRIBUTE"));
        return false;
    }
    if (item2->GetAttributeCount() < 3) {
        if (number(1, 100) < 30) {
            while(item2->GetAttributeCount() < number(2, 3))
                item2->AddAttribute();
        }
    }
}
// Enchant costume
if (item->GetVnum() == 70064) {
    if (item2->GetType() != ITEM_COSTUME) {
        ChatPacket(CHAT_TYPE_INFO, LC_TEXT("ITEM_ISNT_COSTUME"));
        return false;
    }
    if (item2->GetAttributeCount() == 0) {
        ChatPacket(CHAT_TYPE_INFO, LC_TEXT("ITEM_HASNT_ATTRIBUTE"));
        return false;
    }
}
```

---

## 🎮 Clientseitige Änderungen

### 1. `uiinventory.py`

**Suche:**
```python
elif "USE_CHANGE_ATTRIBUTE" == useType:
```

**Ersetzen mit:**
```python
elif "USE_CHANGE_ATTRIBUTE" == useType:
    if srcItemVNum == 70063 or srcItemVNum == 70064:
        if self.__CanChangeCostumeAttrList(dstSlotPos):
            return True
    else:
        if self.__CanChangeItemAttrList(dstSlotPos):
            return True
```

**Suche nach:**
```python
def __CanPutAccessorySocket(self, dstSlotPos, mtrlVnum):
```

**Und direkt darüber einfügen:**
```python
def __CanChangeCostumeAttrList(self, dstSlotPos):
    dstItemVNum = player.GetItemIndex(dstSlotPos)
    if dstItemVNum == 0:
        return False

    item.SelectItem(dstItemVNum)

    if item.GetItemType() == item.ITEM_TYPE_COSTUME:
        return True

    return False
```

---

## 🗃️ Weitere Dateien

### `item_proto.txt` (Server)
```
70063	transform_costume	ITEM_USE	USE_CHANGE_ATTRIBUTE	...
70064	enchant_costume	ITEM_USE	USE_CHANGE_ATTRIBUTE	...
```

### `item_names.txt` (Server)
```
70063	Transform Costume
70064	Enchant Costume
```

### `itemdesc.txt` (Client)
```
70063	Transform Costume	Removes the bonuses from one of your costumes and replaces them with new ones. With a bit of luck, the number of bonuses might also change (up to max. 3).
70064	Enchant Costume	Removes the bonuses from one of your costumes and replaces them with new ones.
```

### `item_list.txt` (Client)
```
70063	ETC	icon/item/70063.tga	
70064	ETC	icon/item/70064.tga
```

