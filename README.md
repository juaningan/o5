
# Table of Contents

1.  [Overview](#org763e369)
2.  [Report Context](#org57c2d6c)
    1.  [Overview](#org17e0de3)
    2.  [data](#orgc38b341)
    3.  [workflows](#orgbb42c99)
3.  [Fishing Area Context](#orgcbf157f)
    1.  [Overview](#orgd6ef2c7)
    2.  [Goals](#org70dfdc8)
    3.  [data](#org9c8b8f0)
    4.  [workflows](#org7c18cc9)
        1.  [substeps](#orga91ecc0)
4.  [Vessel Identification Context](#org9eb98e4)
    1.  [Overview](#org14194d0)
    2.  [Goals](#org21b441f)
    3.  [data](#org2307bce)
5.  [Informer Data Context](#org92cc570)
    1.  [Overview](#org5ac335e)
    2.  [Goals](#org75b4bb4)
    3.  [data](#orgb79c478)
6.  [Help Context](#orgf2de306)
    1.  [Overview](#org118bd1e)
    2.  [data](#orga0cabe7)
7.  [Shareable Report Context](#org6ba0e62)
    1.  [data](#orgc5437e4)
8.  [Local authorities Report Context](#org39b862b)
    1.  [data](#orgf5f8156)
9.  [Public Report Context](#orgf467d3a)
    1.  [data](#org3d33352)
10. [SPRFMO Report Context](#org3ffd70e)
    1.  [data](#org7960226)



<a id="org763e369"></a>

# Overview

![img](components-app.png)


<a id="org57c2d6c"></a>

# Report Context


<a id="org17e0de3"></a>

## Overview

High sees encounters with alleged ilegal fishing vessels (mainly Asian flagged
sheeps) are being reported by artisanal fishing fleet. They usually
don’t have the tools to record those encounters properly, and there is lack of
agreement on what information should be registered.

The encounters usually take place in non network coverage areas, crusing up to 8 days


<a id="orgc38b341"></a>

## data

    data Report =
      FishingVessel
      AND NoEmptyList of SuspiciousActivity
      AND InformerData


<a id="orgbb42c99"></a>

## workflows

    workflow "ReportSuspiciousActivity" =
      input = EmptyReport
      output (on success) = SuspiciousActivityReported
      output (on error) = InvalidReport
    
    //step 1
    do CheckFields


<a id="orgcbf157f"></a>

# Fishing Area Context


<a id="orgd6ef2c7"></a>

## Overview

The project is aimed towards ​​Calamasur. The boundaries of the Exclusive Economic
Zones (EEZ) where the greatest number of encounters take place are defined and
accepted internationally.

Other fishing areas may have totally or subtly different problems from  Calamasur ones, and for which SFP does not currently have agreements or powers. Future versions supporting those concerns should not be discarded. 

All devices are able to obtain their geolocation.


<a id="org70dfdc8"></a>

## Goals

-   Improve the user experience with an interface and business rules adapted to
    the location.
-   Verify device's location capabilities are turned on.
-   Ensure that at the beginning of the process we have a pretty accurate
    position.
-   Minimize the number of non valid reports that will need oversight (From land,
    areas that are not taken in consideration)


<a id="org9c8b8f0"></a>

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


<a id="org7c18cc9"></a>

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


<a id="orga91ecc0"></a>

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


<a id="org9eb98e4"></a>

# Vessel Identification Context


<a id="org14194d0"></a>

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


<a id="org21b441f"></a>

## Goals

-   Get the most relevant information possible
-   Get alerts prior to obtaining sound and relevant information
-   Educate about the different reportable facts, other than the fishing activity.
-   Try to obtain graphic evidence, even of low quality, where details of the
    construction of the ships are captured.
-   Get to know if the AIS system is non-operative or sending false data
    (spoofing)


<a id="org2307bce"></a>

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


<a id="org92cc570"></a>

# Informer Data Context


<a id="org5ac335e"></a>

## Overview

Standard user (Informer) would ususally be an artisanal fish skipper, resistant
to visibilize his identity or make publicly accessible the accurate point of the
encounter.

Artisanal fishing fleet does not have AIS transmitors, implementing VMS system,
while industrial fishing fleet they do transmit AIS publicly.

Android devices can dump raw GNSS data.


<a id="org75b4bb4"></a>

## Goals

-   Draw as much as possible information of the moment of the encounter
-   Truthfulness needed to validate the encounter based on AIS and/ or VMS signals
-   Validate technical quality of GNSS signals of the devices


<a id="orgb79c478"></a>

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


<a id="orgf2de306"></a>

# Help Context


<a id="org118bd1e"></a>

## Overview

Application users doesn't know well the law enforcement. Neither mandatory
markings, important markings or international radio systems for indutrial
vessels.

Opportunities to take good pictures are low because mobile device camera
technical specs but also distance, movement, dirty hulls&#x2026;


<a id="orga0cabe7"></a>

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


<a id="org6ba0e62"></a>

# Shareable Report Context


<a id="orgc5437e4"></a>

## data

    data ShareableReport =
      MainPhotograph
      AND ImpreciseLocationMap
      AND ImpreciseHourAndDate
      AND EEZCountryFlag
      AND list of SuspiciousActivity
      AND list of Badge
    
    data Badge =
      PreciseGPSLocation
      OR UserVerified
      OR HighRiskScoreVessel
      OR SPRFMOUnauthorizedVessel
    
    data SuspiciousActivity =
      ...


<a id="org39b862b"></a>

# Local authorities Report Context


<a id="orgf5f8156"></a>

## data

    data LocalAuthoritiesReport =
      FactsDescription
      AND list of AssociatedDocuments
    
    data AssociatedDocuments =
      Photographs
      OR DigitalPictureCertificates
      OR OtherDocuments


<a id="orgf467d3a"></a>

# Public Report Context


<a id="org3d33352"></a>

## data

    data ValidatedReport =
      ValidatedFishingVessel
      AND ValidatedSuspiciousActivity
      AND ValidatedInformerData


<a id="org3ffd70e"></a>

# SPRFMO Report Context


<a id="org7960226"></a>

## data

    data SPRFMOReportingForm =
      DetailsOfVessel
      AND list of ElementContravened
      AND list of AssociatedDocuments
      AND list of RecommendedActions
    
    data DetailsOfVessel =
      Name optional
      AND Flag optional
      AND list of VesselIdentifier
      AND Photographs
      AND Position
      AND SummaryOfAllegedIUUActivities
      AND SummaryOfActions optional
      AND OutcomeOfActions optional
    
    data VesselIdentifier =
      CallSign
      OR IMONumber
      OR OtherVesselIdentifier
    
    data ElementContravened =
      NotRegisteredOnSPRFMOList
      OR FishingInClosedAreas
      OR UseNonCompliantFishingGear
      OR FishingWithoutNationality
    
    data AssociatedDocuments =
      Photographs
      OR DigitalPictureCertificates
      OR AttachedDocuments

