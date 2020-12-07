# TornadoPackageServer

# Common concepts of Tornado Package Server (TPS)

## Implementation variants

- TornadoPackageServer (C#) <--> TornadoEditor (C++)
- TornadoPackageServer (C++) <--> TornadoEditor (C++)

## Tech stack

- PostgreSQL
- Transport (TCP)
- RSA and AES encryption
- Sending mail
- JSON serialization

Object -> JSON -> PacketObject -> JSON -> Compression -> AES -> TCP packet

All packets are in JSON format.  

``` cpp
struct Packet
{  
    int16 packetId;  
    string body;
};
```

## Commands

- RegisterUser            - {Email} -> {ResultCode}  
- CreatePackage           - {name, info}          -> {ResultCode}  
- DeletePackage           - {PackageName}         -> {ResultCode}
- UpdatePackageInfo       - {PackageName, info}   -> {ResultCode}
- UpdatePackageProjectUrl - {PackageName, url}    -> {ResultCode}
- UpdatePackageScreenUrl  - {PackageName, url}    -> {ResultCode}
- UpdatePackageType       - {PackageName, type}   -> {ResultCode}
- FindPackage             - {PackageName}         -> {[PackageName]}
- FindPackagePageCount    - {PackageName}         -> {PageCount}
- FindPackagePage         - {PackageName, PageNumber, Sorting}  
                           -> [{PackageName, Info, Rating, RatingCount, ProjectUrl, ScreenUrl, UserName, Type, [PackageVersionName, DateTime, Rating, RatingCount, DownloadCount]}]
- GetPackageCount         - {}                    -> {PageCount}
- GetPackageList          - {PageNumber, Sorting}  
                           -> [{PackageName, Info, Rating, RatingCount, ProjectUrl, ScreenUrl, UserName, Type, [PackageVersionName, DateTime, Rating, RatingCount, DownloadCount]}]
- RatePackage             - {PackageName, Rating} -> {ResultCode}

- AddPackageVersion                - {PackageName, Name}                         -> {ResultCode}
- BeginUploadPackageVersionContent - {PackageName, PackageVersionName, [uint8_t]}   -> {ResultCode}
- EndUploadPackageVersionContent   - {PackageName, PackageVersionName}           -> {ResultCode}
- DeletePackageVersion             - {PackageName, PackageVersionId}             -> {ResultCode}
- UpdatePackageVersionType         - {PackageName, PackageVersionName, Type}     -> {ResultCode}
- AddCanGetPackageVersion          - {PackageName, PackageVersionName, UserName} -> {ResultCode}
- DeleteCanGetPackageVersion       - {PackageName, PackageVersionName, UserName} -> {ResultCode}
- GetPackageVersionPageCount       - {PackageName, PackageVersionName}           -> {PageCount}
- GetPackageVersionPage            - {PackageName, PackageVersionName,PageNumber}-> {PageNumber,[uint8_t]}
- RatePackageVersion               - {PackageName, PackageVersionName, Rating}   -> {ResultCode}

---
Sorting {ByPackageRatingUp, ByPackageRatingDown, ByPackageNameUp, ByPackageNameDown, ByDownloadCountUp, ByDownloadCountDown, ByUserNameUp, ByUserNameDown}
ResultCode {Ok, Fail, EmailIsExist}

### Tables

``` pascal
User
- Id uid [unique]
- Name string [unique]
- Email string [unique]
- PasswordHash string
- Permit uint8 {0-None, 1-PackageOperation}
```

``` pascal
Package
- Id uid unique
- Name string unique
- MasterUserId uid
- Info string
- PackageType uint8 {0-Private, 1-Public}
- ProjectUrl string
- ScreenUrl string
```

``` pascal
PackageVersion
- Id uid unique
- Name string
- PackageId uid
- DateTime DateTime
- Content Binary
```

``` pascal
PackageVersionAccess
- Id uid unique
- UserId uid
- PackageId uid
- PackageVersionId uid
- AccessLevel uint8 = Read
```

``` pascal
PackageRating
- Id uid unique
- PackageId uid
- UserId uid
- Rating int8
```

``` pascal
PackageVersionRating
- Id uid unique
- PackageId uid
- PackageVersionId uid
- UserId uid
- Rating int8
```

``` pascal
PackageVersionDownloadStat
- Id uid unique
- PackageId uid
- PackageVersionId uid
- UserId uid
- DateTime DateTime
```
