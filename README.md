
# Table of Contents

1.  [Overview](#org3937077)
2.  [Report Context](#orgcafcdfc)
    1.  [Overview](#orgb4326df)
    2.  [data](#orge181158)
    3.  [workflows](#org3694901)
3.  [Fishing Area Context](#org34cb186)
    1.  [Overview](#org3546577)
    2.  [Goals](#org83276b8)
    3.  [data](#org44979cd)
    4.  [workflows](#org7fe64ff)
        1.  [substeps](#org38c45ca)
4.  [Vessel Identification Context](#org9bc23fa)
    1.  [Overview](#orgedd9751)
    2.  [Goals](#org49b1e49)
    3.  [data](#org7214463)
5.  [Informer Data Context](#org2efff73)
    1.  [Overview](#org711eb8b)
    2.  [Goals](#org892c964)
    3.  [data](#orgdb03861)
6.  [Help Context](#orgb40ca1a)
    1.  [Overview](#orgd0da390)
    2.  [data](#org75f2b52)



<a id="org3937077"></a>

# Overview

![img](components-app.png)


<a id="orgcafcdfc"></a>

# Report Context


<a id="orgb4326df"></a>

## Overview

High sees encounters with alleged ilegal fishing vessels (mainly Asian flagged
sheeps) are being reported by artisanal fishing fleet. They usually
don’t have the tools to record those encounters properly, and there is lack of
agreement on what information should be registered.

The encounters usually take place in non network coverage areas, crusing up to 8 days


<a id="orge181158"></a>

## data

    data Report =
      FishingVessel
      AND NoEmptyList of SuspiciousActivity
      AND InformerData


<a id="org3694901"></a>

## workflows

    workflow "ReportSuspiciousActivity" =
      input = EmptyReport
      output (on success) = SuspiciousActivityReported
      output (on error) = InvalidReport
    
    //step 1
    do CheckFields


<a id="org34cb186"></a>

# Fishing Area Context


<a id="org3546577"></a>

## Overview

The project is aimed towards ​​Calamasur. The boundaries of the Exclusive Economic
Zones (EEZ) where the greatest number of encounters take place are defined and
accepted internationally.

Other fishing areas may have totally or subtly different problems from  Calamasur ones, and for which SFP does not currently have agreements or powers. Future versions supporting those concerns should not be discarded. 

All devices are able to obtain their geolocation.


<a id="org83276b8"></a>

## Goals

-   Improve the user experience with an interface and business rules adapted to
    the location.
-   Verify device's location capabilities are turned on.
-   Ensure that at the beginning of the process we have a pretty accurate
    position.
-   Minimize the number of non valid reports that will need oversight (From land,
    areas that are not taken in consideration)


<a id="org44979cd"></a>

## data

    data DeviceLocationData =
      Latitude
      AND Longitude
      AND Accurancy
    
    data DesiredAccurancy =
      Meters
    
    data GetFishingAreaInput =
      DeviceLocationData
      AND DesiredAccurancy
    
    data AccurateLocation =
      AccurateLatitude
      AND AccurateLongitude
    
    data SupportedAreas =
      list of SupportedArea
    
    data SupportedArea =
      FishingAreaName
      AND PolygonArea // As in marineregions.org
    
    data SupportedFishingArea =
      CalamasurArea
      OR GenericArea
      OR ...


<a id="org7fe64ff"></a>

## workflows

    workflow "GetFishingArea" =
      input: GetFishingAreaInput
      output (on success): SupportedFishingAreaDetected
      output (on error): UnsupportedFishingArea OR InaccurateLocation
    
    //step1
    do CheckLocationAccurancy
      If inaccurate
      return InaccurateLocation
      stop
    
    //step2
    do CheckPosition
      If unsupported
      return UnsupportedFishingArea
      stop
    
    //step3
    return SupportedFishingAreaDetected


<a id="org38c45ca"></a>

### substeps

    substep "CheckLocationAccuracy" =
      input: GetFishingAreaInput
      output (on success): AccurateLocation
      output (on error): InaccurateLocation
    
    substep "CheckPosition" =
      input: AccurateLocation
      dependency: SupportedAreas
      output (on sucess): SupportedFishingArea
      output (on error): UnsupportedFishingArea


<a id="org9bc23fa"></a>

# Vessel Identification Context


<a id="orgedd9751"></a>

## Overview

In Calamasur, the alleged illegal fishing vessels are mainly of Asian flag.

It can be particularly difficult to identify Asian flagged ships by their names, since
it exists a great variation in the way Chinese characters are written in Roman
letters, therefore it is mandatory to detect any numbers visible on the hull.

There is lack of consensus regarding the proximity of the encounters and if it
is possible to capture relevant information with a mobile device. However, it
seems unanimous that suspicious vessels are usually very dirty, show unreadable
names and attempts to hide their features.

According to FAO, attempts to hide their features are already a felony,
regardless of the fishing activity they are carrying out same for duplicate
features, or sailing without a flag.

Suspicious vessels carry out an industrial activity, so they are obliged to
broadcast through the AIS radio system. However, the majority of Calamasur craft
vessels do not have AIS receivers whereas the industrial fleets in the area do
have them.


<a id="org49b1e49"></a>

## Goals

-   Get the most relevant information possible
-   Get alerts prior to obtaining sound and relevant information
-   Educate about the different reportable facts, other than the fishing activity.
-   Try to obtain graphic evidence, even of low quality, where details of the
    construction of the ships are captured.
-   Get to know if the AIS system is non-operative or sending false data
    (spoofing)


<a id="org7214463"></a>

## data

    // Markings
    data IMONumber =
      string 9 numbers
    
    data CallSign =
      string first 3 chars as ITU codes
    
    data Other =
      string
    
    data Marking <a> =
      <a>
      OR <a> MarkingNotVisibleOrHidden
    
    data Markings =
      Marking<CallSign>
      AND Marking<IMONumber>
      AND list of Marking<Others>
    
    
    // AIS Transmission
    data MMSI =
      string 7 numbers
    
    data AISDeclaredActivity =
      Fishing
      OR ToPort
      OR ...
    
    data AISTransmission =
      MMSI
      AND AISCallSign
      AND AISName
      AND AISDeclaredActivity
    
    
    // Vessel
    data FishingVessel =
      Type
      AND Markings
      AND AISTransmission
      AND list of Photos
    
    // Activity
    data SuspiciousActivity =
      EEZFishingActivity
      OR HiddingMarkings
      OR DuplicatedMarkings
      OR SignsOfTampering
      OR Stateless
      OR NotAISSignal
      OR AISSpoofedSignal
      OR Other


<a id="org2efff73"></a>

# Informer Data Context


<a id="org711eb8b"></a>

## Overview

Standard user (Informer) would ususally be an artisanal fish skipper, resistant to visibilize his identity or make publicly accessible the accurate point of the encounter.

Artisanal fishing fleet does not have AIS transmitors, implementing VMS system, while industrial fishing fleet they do transmit AIS publicly.

Android devices can dump raw GNSS data.


<a id="org892c964"></a>

## Goals

-   Draw as much as possible information of the moment of the encounter
-   Truthfulness needed to validate the encounter based on AIS and/ or VMS signals
-   Validate technical quality of GNSS signals of the devices


<a id="orgdb03861"></a>

## data

    data PeruvianVMSIdentifier =
      String as "P-04-00951"
    
    data ChileanVMSIdentifier =
      VesselName
    
    data VMSIdentifier =
      PeruvianVMSId
      OR ChileanVMSId
    
    data InformerDeviceLocation =
      RawGNSSMeasurementsDump
      OR BasicLocation
    
    data InformerAIS =
      MMSINumber
      OR WithoutOrPrivateAIS
    
    data InformerVMS =
      VMSIdentifier
      OR WithoutOrPrivateVMS
    
    data InformerData =
      InformerVMS
      AND InformerAIS
      AND InformerDeviceLocation


<a id="orgb40ca1a"></a>

# Help Context


<a id="orgd0da390"></a>

## Overview

Application users doesn't know well the law enforcement. Neither mandatory
markings, important markings or international radio systems for indutrial
vessels.

Opportunities to take good pictures are low because mobile device camera
technical specs but also distance, movement, dirty hulls&#x2026;


<a id="org75f2b52"></a>

## data

    data MarkingsHelp =
      Name
      AND Description
      AND list of Examples
      AND list of TypicalLocation
      AND list of ExamplePhotos
    
    data TypicalLocation =
      Stern
      OR Side
      OR Bridge

