---
layout: publications
title: "Automated xCode project builds from terminal."
categories: publications
modified: 2013-02-20T11:57:41-04:00
toc: false
comments: true
---

100% working bash compilation commands for xCode 4.x+
I spent about 3 weeks in google+bash+xcode to make everything work !!! It was a kind of insane and its finally done! I hope this could help someone.

##keychain & AppleWWDRCA.cer

{% highlight bash %}
#settings
USER=$1;
PASSWORD=$2
P12=$3
MOB=$4

echo ========================
echo USER = $USER;
echo PASSWORD = $PASSWORD;
echo P12 = $P12;
echo MOB = $MOB;
echo ========================



security unlock-keychain -p 12qw12 $USER.keychain
security import $P12 -P $PASSWORD -k $USER.keychain -A
security add-certificates -k $USER.keychain /data/AppleWWDRCA.cer

#cp $MOB ~/Library/MobileDevice/Provisioning Profiles/
{% endhighlight %}

##xCode build: Development
{% highlight bash %}
USER=$1
IDENTITY=$2
TARGET="gomobile001"
CONFIGURATION="Debug"
USERDIR=/data/users/$USER

whoami
echo ========================
echo USER = $USER;
echo USERDIR = $USERDIR;
echo IDENTITY = $IDENTITY;
echo CONFIGURATION = $CONFIGURATION;
echo ========================

function deleteKeychain() {
security delete-keychain xcodebuild.keychain
security default-keychain -s login.keychain
}

function createKeychain() {
security create-keychain -p 12qw12 xcodebuild.keychain
security add-certificates -k xcodebuild.keychain /data/AppleWWDRCA.cer
security unlock-keychain -p 12qw12 xcodebuild.keychain
security import $USERDIR/dev.p12 -P 12qw12 -k xcodebuild.keychain -A
security default-keychain -s xcodebuild.keychain
}

cp $USERDIR/dev.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
cd /data/$TARGET/

createKeychain;

xcodebuild -target "$TARGET" -configuration "$CONFIGURATION" CODE_SIGN_IDENTITY="$IDENTITY" OTHER_CODE_SIGN_FLAGS="--keychain xcodebuild.keychain" CONFIGURATION_BUILD_DIR="$USERDIR"
/usr/bin/xcrun -sdk iphoneos PackageApplication -v "$USERDIR/$TARGET.app" -o "$USERDIR/$TARGET.ipa" --embed "$USERDIR/dev.mobileprovision"

deleteKeychain;

exit
{% endhighlight %}

##xCode build: Production

{% highlight bash %}
USER=$1
IDENTITY=$2
TARGET="gomobile001"
CONFIGURATION="Release"
USERDIR=/data/users/$USER

whoami
echo ========================
echo USER =  $USER;
echo USERDIR =  $USERDIR;
echo IDENTITY = $IDENTITY;
echo CONFIGURATION = $CONFIGURATION;
echo ========================


function deleteKeychain() {
    security delete-keychain xcodebuild.keychain
    security default-keychain -s login.keychain
}

function createKeychain() {
    security create-keychain -p 12qw12 xcodebuild.keychain
    security add-certificates -k xcodebuild.keychain /data/AppleWWDRCA.cer
    security unlock-keychain -p 12qw12 xcodebuild.keychain
    security import $USERDIR/prod.p12 -P 12qw12 -k xcodebuild.keychain -A
    security default-keychain -s xcodebuild.keychain
}

cp $USERDIR/prod.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
cd /data/$TARGET/


createKeychain;

xcodebuild -target "$TARGET" -configuration "$CONFIGURATION" CODE_SIGN_IDENTITY="$IDENTITY" OTHER_CODE_SIGN_FLAGS="--keychain xcodebuild.keychain" CONFIGURATION_BUILD_DIR="$USERDIR"
/usr/bin/xcrun -sdk iphoneos PackageApplication -v "$USERDIR/$TARGET.app" -o "$USERDIR/$TARGET.ipa" --embed "$USERDIR/prod.mobileprovision"

deleteKeychain;

exit
{% endhighlight %}
