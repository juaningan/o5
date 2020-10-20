
# Table of Contents

1.  [Overview](#orgf5b3b35)
2.  [Report Context](#orgab20367)
    1.  [Overview](#org5091983)
    2.  [data](#orgda1675c)
    3.  [workflows](#org99f82a0)
3.  [Fishing Area Context](#org7bde16c)
    1.  [Overview](#org478290d)
    2.  [Goals](#org59718cd)
    3.  [data](#orgdcd508b)
    4.  [workflows](#org481d4e1)
        1.  [substeps](#orgdc44b24)
4.  [Vessel Identification Context](#org0b792b1)
    1.  [Overview](#org8fb33e2)
    2.  [Goals](#orgfa03a51)
    3.  [data](#org7e2b786)
5.  [Informer Data Context](#org482c868)
    1.  [Overview](#org269deec)
    2.  [Goals](#orga8a31bc)
    3.  [data](#org8422b4c)
6.  [Help Context](#org5b8538b)
    1.  [Overview](#orga4539ec)
    2.  [data](#org68d36c6)
7.  [Shareable Report Context](#orgd4e89ad)
    1.  [data](#orgf115e57)
8.  [Local authorities Report Context](#org67c8938)
    1.  [data](#orgdcd9a1e)
9.  [Public Report Context](#orga030660)
    1.  [data](#org9e3fd2d)
10. [SPRFMO Report Context](#org291746c)
    1.  [data](#orga007ddd)



<a id="orgf5b3b35"></a>

# Overview

![img](components-app.png)


<a id="orgab20367"></a>

# Report Context


<a id="org5091983"></a>

## Overview

High sees encounters with alleged ilegal fishing vessels (mainly Asian flagged
sheeps) are being reported by artisanal fishing fleet. They usually
don’t have the tools to record those encounters properly, and there is lack of
agreement on what information should be registered.

The encounters usually take place in non network coverage areas, crusing up to 8 days


<a id="orgda1675c"></a>

## data

    data Report =
      FishingVessel
      AND NoEmptyList of SuspiciousActivity
      AND InformerData


<a id="org99f82a0"></a>

## workflows

    workflow "ReportSuspiciousActivity" =
      input = EmptyReport
      output (on success) = SuspiciousActivityReported
      output (on error) = InvalidReport
    
    //step 1
    do CheckFields


<a id="org7bde16c"></a>

# Fishing Area Context


<a id="org478290d"></a>

## Overview

The project is aimed towards ​​Calamasur. The boundaries of the Exclusive Economic
Zones (EEZ) where the greatest number of encounters take place are defined and
accepted internationally.

Other fishing areas may have totally or subtly different problems from  Calamasur ones, and for which SFP does not currently have agreements or powers. Future versions supporting those concerns should not be discarded. 

All devices are able to obtain their geolocation.


<a id="org59718cd"></a>

## Goals

-   Improve the user experience with an interface and business rules adapted to
    the location.
-   Verify device's location capabilities are turned on.
-   Ensure that at the beginning of the process we have a pretty accurate
    position.
-   Minimize the number of non valid reports that will need oversight (From land,
    areas that are not taken in consideration)


<a id="orgdcd508b"></a>

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


<a id="org481d4e1"></a>

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


<a id="orgdc44b24"></a>

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


<a id="org0b792b1"></a>

# Vessel Identification Context


<a id="org8fb33e2"></a>

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


<a id="orgfa03a51"></a>

## Goals

-   Get the most relevant information possible
-   Get alerts prior to obtaining sound and relevant information
-   Educate about the different reportable facts, other than the fishing activity.
-   Try to obtain graphic evidence, even of low quality, where details of the
    construction of the ships are captured.
-   Get to know if the AIS system is non-operative or sending false data
    (spoofing)


<a id="org7e2b786"></a>

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


<a id="org482c868"></a>

# Informer Data Context


<a id="org269deec"></a>

## Overview

Standard user (Informer) would ususally be an artisanal fish skipper, resistant
to visibilize his identity or make publicly accessible the accurate point of the
encounter.

Artisanal fishing fleet does not have AIS transmitors, implementing VMS system,
while industrial fishing fleet they do transmit AIS publicly.

Android devices can dump raw GNSS data.


<a id="orga8a31bc"></a>

## Goals

-   Draw as much as possible information of the moment of the encounter
-   Truthfulness needed to validate the encounter based on AIS and/ or VMS signals
-   Validate technical quality of GNSS signals of the devices


<a id="org8422b4c"></a>

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


<a id="org5b8538b"></a>

# Help Context


<a id="orga4539ec"></a>

## Overview

Application users doesn't know well the law enforcement. Neither mandatory
markings, important markings or international radio systems for indutrial
vessels.

Opportunities to take good pictures are low because mobile device camera
technical specs but also distance, movement, dirty hulls&#x2026;


<a id="org68d36c6"></a>

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


<a id="orgd4e89ad"></a>

# Shareable Report Context


<a id="orgf115e57"></a>

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


<a id="org67c8938"></a>

# Local authorities Report Context


<a id="orgdcd9a1e"></a>

## data

    data LocalAuthoritiesReport =
      FactsDescription
      AND list of AssociatedDocuments
    
    data AssociatedDocuments =
      Photographs
      OR DigitalPictureCertificates
      OR OtherDocuments


<a id="orga030660"></a>

# Public Report Context


<a id="org9e3fd2d"></a>

## data

\#+BEGINS<sub>SRC</sub>
data ValidatedReport =
  ValidatedFishingVessel
  AND ValidatedSuspiciousActivity
  AND ValidatedInformerData
\#+END<sub>SRC</sub>


<a id="org291746c"></a>

# SPRFMO Report Context


<a id="orga007ddd"></a>

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

