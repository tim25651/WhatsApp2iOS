# WhatsApp2iOS
This tutorial should help you to move your WhatsApp messages
from a Android phone to 
As part of the process the iPhone has to be back upped and restored again.
Thus, the earlier in the setting up of the iPhone the more convenient.

## Prerequisites
- An unencryped message database (Skip #2 and #3),  
  or python/conda, an encrypted message database and your WhatsApp key file (Skip #2),  
  or a rooted Android phone with activated USB debugging and ADB

- Conda or Python with pip
- XCode
- An .ipa file of WhatsApp in the same version
- Apple Configurator/iTunes/Finder

 
## Steps

1. Prepare workspace  
```
mkdir WhatsApp2iOS
export WA2IOS=$(PWD)/WhatsApp2iOS
cd $WA2IOS
git clone https://github.com/ElDavoo/WhatsApp-Crypt14-Decrypter
git clone https://github.com/residentsummer/watoi
```


2. Retrieve the data from your source device  
Extract the encryption key from /data/data/com.whatsapp/files/key with:
```
adb root
adb -d shell "cat data/data/com.whatsapp/files/key" > $W2IOS/key
```
Pull the latest message database from WhatsApp
```
adb pull /sdcard/Android/media/com.whatsapp/WhatsApp/Databases/msgstore.db.crypt14 $W2IOS
```

3. Decrypt WhatsApp database  
Set up the python environment with conda:
```
conda create -n wa2ios pycrypto pycryptodome
conda activate wa2ios
```
or with pip:
```
pip install pycrypto pycryptodome
```
Decrypt the .crypt14 file
```
cd $WA2IOS
python WhatsApp-Crypt14-Decrypter/decrypt14.py key msgstore.db.crypt14 msgstore.db
```

4. Prepare iPhone  
Install WhatsApp from iPhone from the App Store, and activate with the same phone number as the Android phone  
You should see now a list of all of your group chats  
Create a backup with for example the Apple Configurator 2:  
Select your iPhone under Unsupervised, and select Actions -> Backup (Remember the creation time  
Give your terminal app full storage access:  
Command+Space -> Spotlight -> "Security & Privacy" -> Privacy -> Full Disk Access -> Check iTerm (e.g.)  
Show all iPhone backups, store the ID of the created backup in a variable and create a safety backup:
```
cd $WA2IOS
ls -l  /Users/tim/Library/Application\ Support/MobileSync/Backup
export BACKUP_ID="YOUR BACKUP ID" # Example: 00000000-444408800C04422E
cp -R /Users/tim/Library/Application\ Support/MobileSync/Backup/$(BACKUP_ID) backup
```


5. Extract the messages from the database  
```
cd $WA2IOS
export ORIGINALS="originals/$(date +%s)"
mkdir -p $ORIGINALS
watoi/scripts/bedit.sh extract-chats $BACKUP_ID $ORIGINALS/ChatStorage.sqlite
watoi/scripts/bedit.sh extract-blob $BACKUP_ID Manifest.db $ORIGINALS/Manifest.db
cp $ORIGINALS/ChatStorage.sqlite ./ChatStorage.sqlite
```

6. Patch the backup folder  
Get the CoreData files from the .ipa file, build the WATOI Patcher and run the migration:
```
cd $WA2IOS
unzip ~/Download/WhatsApp_x.x.x.ipa -d app
cd $WA2IOS/watoi
xcodebuild -project watoi/watoi.xcodeproj -target watoi
watoi/build/Release/watoi msgstore.db ./ChatStorage.sqlite app/Payload/WhatsApp.app/Frameworks/Core.framework/WhatsAppChat.momd
```

7. Restore the backup  
Restore the backup with the Apple Configurator:  
Select your iPhone under Unsupervised, and select Actions -> Restore from Backup -> Choose the backup with the earlier remembered creation time  
After the restoring process has finished, reinstall WhatsApp and login with the same phone number
