FORMAT:  1A

HOST: https://localhost/api/

Metasys API
===========

The Metasys API is the exposed interface for accessing system data. This document defines the contracts included in that interface.

<!--
**See HLD for more detailed information about the API**
-->

## API Version

Version: 1  
Prefix : /api/v1

### API Version Notes

<!--
This document describes version 1 of the API (not to be confused with document version in [Document Information](#document-information) section above). -->

While this is currently the only version available, as new versions are added the API consumer will need to specify which version of the API they intend to use.  
The primary means of doing this is to add the version before the first entity in the URL using the format "v[#]" (e.g. the `v1` in `http://localhost/api/v1/spaces`).  
If using the URL is undesirable, an alternate approach is to use the `accept` header in the HTTP request, and modify the version portion based on which version is intended (e.g. the `v1` in `application/vnd.metasysapi.v1+json`).  
The version (if used) in the URL will always override the version (if used) in the header. If no version is specified, the lowest supported version (version 1) will be used. If an invalid version is specified, a 404 (not found) error status will be returned.  
The `Content-Type` header of the response will always contain (in part) the version of the API that was used to serve the request, so the consumer can use this to identify which version of the API was utilized.

## Pagination

On endpoints where `page` and `pageSize` is allowed, the default `page` number will be 1 and is 1-based for all endpoints while the default `pageSize` will vary between endpoints.  The `page` parameter indicates the `page` number of items to return from the endpoint.  The `pageSize` parameter indicates the maximum number of items in the response from the endpoint.

Payloads returned by pagination-enabled endpoints will have a similar structure. A `total` property will indicate the total number of items included in all pages. A `next` and `previous` property will supply a link to the next and previous page of data, respectively (These properties can be empty if irrelevant, e.g. it is the first/last page, or there is only one page of data). The `items` property contains the data included in the page.

## Sorting Rules

On endpoints where a `sort` query parameter is allowed, the supplied value should be in the format of a single attribute name, optionally prefixed with `-` to indicate descending sort order (ascending order is used if no prefix is supplied).

## Relationship Links

Payloads may contain links to related data, each represented as a property sharing the name of the respective relationship. The links point to either single or multiple related entities. A link to a single entity points to the cardinal endpoint for that entity. A link to multiple entities points to an endpoint dedicated to representing that particular relationship.

For example, if object `/objects/a` has children `/objects/b` and `/objects/c`, the payload returned by `/objects/a` will have a property `hasChildren` with a value of `/objects/a/hasChildren`, because multiple children can be returned. However, the payload returned by `/objects/b/` will contain a property 'isChildOf' with a value of `/objects/a` (not `/objects/b/isChildOf`), because the relationship represents a single entity.

Additionally, each payload will contain a `self` property, which contains a link representing the endpoint used to obtain the data contained in the current payload.

## Validation

There are some general rules that apply across all endpoints. If certain provided inputs are invalid or preconditions are not met, the API wll respond with an appropriate error to indicate what went wrong.

|Condition                      |Error             |Details                                                                                                    |
|-------------------------------|------------------|-----------------------------------------------------------------------------------------------------------|
|User not authenticated         |401 (Unauthorized)|The auth token supplied with the request is missing, invalid, or expired                                   |
|Record not authorized          |403 (Forbidden)   |The user is not authorized to view data matching the provided identifier                                   |
|Identifier not found           |404 (Not Found)   |An identifier is provided that does not match any known data                                               |
|Missing required parameter     |400 (Bad Request) |A parameter marked required in this document is not included in the request                                |
|Parameter incorrect type       |400 (Bad Request) |A parameter is included with a value of the wrong type (e.g. number is expected and string is provided)    |
|Parameter out of range         |400 (Bad Request) |A numeric parameter is included but the value is outside the allowed range                                 |
|Parameter not in set           |400 (Bad Request) |A string parameter has a set of predefined valid values, and the value provided is not included in that set|
|Parameter not in correct format|400 (Bad Request) |A string parameter with expected format is provided in the wrong format                                    |

<!-- include(include.md) -->

Group Activities
================

This section contains information about activities as used within the Metasys API

## Activities Collection [/activities]

### Get Activities [GET /activities/{?originApplications,classLevels,actions,ids,startTime,endTime,page,pageSize,sort,excludeDiscarded}]
Retrieves a collection of activities. When using multiple filters, an AND evaluation is assumed.

+ Parameters
    + originApplications:`1,2` (string, optional) - Filter by comma-separated list of origin applications
        + See /enumSets/578/members for possible values
    + classLevels:`1,2` (string, optional) - Filter by comma-separated list of class levels
        + See /enumSets/568/members for possible values
    + actions:`1,2` (string, optional) - Filter by comma-separated list of the actions
        + See /enumSets/577/members for possible values
    + ids:`E1C28F29-90DC-464F-90DE-07DC08454E31,483FB3F7-6C65-4570-9DC2-01F4FD1A8F3A` (string, optional) -  Filter by comma-separated list of identifiers of the audits
    + startTime:`2018-05-21T13:58:20.243Z` (string, optional) - The ISO-8601 encoded dateTime representing the earliest activity to return
    + endTime:`2018-05-22T13:58:20.243Z` (string, optional) - The ISO-8601 encoded dateTime representing the latest activity to return
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `creationTime`
        + Default: `-creationTime`
    + excludeDiscarded (boolean, optional) - Determines whether discarded audits will be excluded from results. Default is `false` (discarded audits will not be excluded).
        + Accepted Values: `true`,`false`
        + Default: `false`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include ActivityEntityCollection
        + self:`/activities{?queryParams}` (string) - A link to this activity data set

## Activity [/activities/{id}]

### Get A Single Activity [GET /activities/{id}]
Retrieves the specified activity.

+ Parameters
    + id:`E1C28F29-90DC-464F-90DE-07DC08454E31` (string) - The identifier of the activity to retrieve
    
+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(ActivityEntity) 

Group Alarms
============

This section contains information about alarms as used within the Metasys API.

## Alarms Collection [/alarms]
A collection of alarms

### Get Alarms [GET /alarms/{?startTime,endTime,priorityMinimum,priorityMaximum,type,excludePending,excludeAcknowledged,excludeDiscarded,attribute,category,page,pageSize,sort}]
Retrieves a collection of alarms.

+ Parameters
    + startTime:`2018-05-21T13:58:20.243Z` (string, optional) - The ISO-8601 encoded dateTime representing the earliest alarm to return
    + endTime:`2018-05-21T13:58:20.243Z` (string, optional) - The ISO-8601 encoded dateTime representing the latest alarm to return
    + priorityMinimum (number, optional) - The minimum priority of the requested alarms
    + priorityMaximum (number, optional) - The maximum priority of the requested alarms 
    + type (number, optional) - The type of the requested alarms
        + See /enumSets/505/members for possible values
    + excludePending (boolean, optional) - The flag to exclude pending alarms
        + Default: `false`
    + excludeAcknowledged (boolean, optional) - The flag to exclude acknowledged alarms
        + Default: `false`
    + excludeDiscarded (boolean, optional) - The flag to exclude discarded alarms
        + Default: `false`
    + attribute (number, optional) - The attribute of the requested alarms
        + See /enumSets/509/members for possible values
    + category (number, optional) - The system category of the requested alarms
        + See /enumSets/33/members for possible values
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `priority`, `creationTime`
        + Default: `creationTime`
    
+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include AlarmEntityCollection
        + self:`/alarms{?queryParams}` (string) - A link to this alarm data set

## Alarm [/alarms/{id}]
A specific alarm in the system

### Get A Single Alarm [GET /alarms/{id}]
Retrieves the specified alarm.

+ Parameters
    + id (string) - The identifier of the alarm

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(AlarmEntity)

### Get Alarm Annotations [GET /alarms/{id}/annotations/{?startTime,endTime,page,pageSize,sort}]
Retrieves the collection of annotations available for the specified alarm.

+ Parameters
    + id (string) - The identifier of the alarm
    + startTime (string, optional) - The ISO-8601 encoded dateTime representing the earliest annotation to return
    + endTime (string, optional) - The ISO-8601 encoded dateTime representing the latest annotation to return
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `creationTime`
        + Default: `creationTime`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include AlarmAnnotationEntityCollection
        + self:`/alarms/{id}/annotations{?queryParams}` (string) - A link to this alarm data set

Group Enumerations
==================

This section contains information about enumeration-related resources as used within the Metasys API.

## Enum Set Collection [/enumSets{?page,pageSize}]
A collection of all enumeration sets in the system.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`

### Get Enum Sets [GET]
Retrieves a collection of all enumeration sets in the system.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EnumSetCollection
        + self:`/enumSets{?queryParams}` (string) - A link to this enum set data set

## Enum Set [/enumSets/{id}]
A set of enumerated values

+ Parameters
    + id (number) - The identifier for the enum set

### Get A Single Enum Set [GET]
Retrieves the specified enumeration set.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(EnumSet)

## Enum Member Collection [/enumSets/{id}/members{?page,pageSize}]
A collection of enumerated member values within an enum set

+ Parameters
    + id (number) - The identifier for the enum set
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`

### Get Enum Members [GET]
Retrieves the collection of member values belonging to the specified enumeration set.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EnumMemberCollection
        + self:`/enumSets/{id}/members{?queryParams}` (string) - A link to this enum member data set

## Enum Member [/enumSets/{id}/members/{memberId}]
A particular member value within an enum set

+ Parameters
    + id (number) - The identifier for the enum set
    + memberId (number) - The identifier for the enum member

### Get A Single Enum Member [GET]
Retrieves the specified enumeration member value.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(EnumMember)

Group Equipment
===============
This section contains information about equipment-related resources as used within the Metasys API.

## Equipment Collection [/equipment{?page,pageSize,sort}]
A collection of Equipment

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `type`
        + Default: `name`

### Get Equipment Instances [GET]
Retrieves a collection of equipment instances.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EquipmentEntityCollection
        + self:`/equipment{?queryParams}` (string) - A link to this equipment data set

## Equipment [/equipment/{id}]
A specific piece of equipment in the system

+ Parameters
    + id (string) - The identifier of the instance of equipment

### Get A Single Equipment Instance [GET]
Retrieves the specified equipment instance.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes (EquipmentEntity)

### Get Spaces Served By An Equipment Instance [GET /equipment/{id}/spacesServed{?page,pageSize,sort}]
Retrieves the collection of spaces served by the specified equipment instance.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include SpaceEntityCollection
        + self:`/equipment/{id}/spacesServed{?queryParams}` (string) - A link to this space data set

### Get Equipment Served By An Equipment Instance [GET /equipment/{id}/equipmentServed{?page,pageSize,sort}]
Retrieves the equipment served by the specified equipment instance.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `type`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EquipmentEntityCollection
        + self:`/equipment/{id}/equipmentServed{?queryParams}` (string) - A link to this equipment data set

### Get Equipment Serving An Equipment Instance [GET /equipment/{id}/servedBy{?page,pageSize,sort}]
Retrieves the collection of equipment that serve the specified equipment instance.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `type`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EquipmentEntityCollection
        + self:`/equipment/{id}/servedBy{?queryParams}` (string) - A link to this equipment data set

### Get Network Devices Hosting An Equipment Instance [GET /equipment/{id}/hostedBy{?page,pageSize,sort}]
Retrieves the collection of network devices that host the specified equipment instance, along with the parents of those network devices. A network device is considered to host an equipment if the equipment defines points that map to an attribute of any object contained on the network device.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `categoryId`, `firmwareVersion`, `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes 
        + Include NetworkDeviceEntityCollection
        + self:`/equipment/{id}/hostedBy{?queryParams}` (string) - A link to this network device data set

### Get Points Defined by An Equipment Instance [GET /equipment/{id}/hasMappings{?page,pageSize,sort}]
Retrieves the collection of points that are defined by the specified equipment instance. Each point contains a mapping to an attribute on an object.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `category`,`equipmentName`, `isDisplayData`, `shortName`
        + Default: `shortName`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes 
        + Include EquipmentPointMappingCollection
        + self:`/equipment/{id}/hasMappings{?queryParams}` (string) - A link to this equipment point mapping data set

Group Network Devices
=====================

This section contains information about network device-related resources as used within the Metasys API

## Network Device Collection [/networkDevices{?type,page,pageSize,sort}]
A collection of all network devices in the system

+ Parameters
    + type (number, optional) - Type of network device to return
        + See /networkDevices/availableTypes for possible values
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `categoryId`, `firmwareVersion`, `itemReference`, `name`, `typeId`
        + Default: `name`

### Get Network Devices [GET]
Retrieves a collection of network devices.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes 
        + Include NetworkDeviceEntityCollection
        + self:`/networkDevices{?queryParams}` (string) - A link to this network device data set

### Get Network Device Types [GET /networkDevices/availableTypes]
Retrieves the collection of all network device types.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include ItemCollection
        + items (array[ObjectTypeEnum], fixed-type)
        + self:`/networkDevices/availableTypes` - A link to this network device type data set

## Network Device [/networkDevices/{id}]
A specific network device in the system

+ Parameters
    + id (string) - The identifier of the network device

### Get A Single Network Device [GET]
Retrieves the specified network device.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(NetworkDeviceEntity)

### Get Network Device Parent [GET /networkDevices/{id}/isChildOf]
Retrieves the network device that is the parent of the specified network device.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(NetworkDeviceEntity)

### Get Network Device Children [GET /networkDevices/{id}/hasChildren{?page,pageSize,sort}]
Retrieves the collection of network devices that are children of the specified network device.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `categoryId`, `firmwareVersion`, `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes 
        + Include NetworkDeviceEntityCollection
        + self:`/networkDevices/{id}/hasChildren{?queryParams}` (string) - A link to this network device data set

### Get Equipment Hosted By A Network Device [GET /networkDevices/{id}/hosts{?page,pageSize,sort}]
Retrieves the collection of equipment instances that are hosted by the specified network device or its children. A network device is considered to host an equipment if the equipment defines points that map to an attribute of any object contained on the network device.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `type`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EquipmentEntityCollection
        + self:` /networkDevices/{id}/hosts{?queryParams}` (string) - A link to this equipment data set

### Get Spaces Hosting A Network Device [GET /networkDevices/{id}/hostedBy{?page,pageSize,sort}]
Retrieves the collection of spaces that host the specified network device. A space is considered to host a network device is considered to host an equipment if any equipment instance serving the space defines points that map to an attribute of any object contained on the network device.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include SpaceEntityCollection
        + self:`/networkDevices/{id}/hostedBy{?queryParams}` (string) - A link to this space data set

### Get Network Device Attributes With Samples [GET /networkDevices/{id}/attributes]
Retrieves a collection of attributes under the specified network device for which samples are available.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include NetworkDeviceAttributeWithSamplesCollection
        + self:`/networkDevices/{id}/attributes` - A link to this data set

### Get Samples For A Network Device Attribute [GET /networkDevices/{id}/attributes/{attributeId}/samples{?startTime,endTime,page,pageSize,sort}]
Retrieves a collection of samples for the specified network device attribute during a particular date and time range.

Note that the parent endpoint `/networkDevices/{id}/attributes/{attributeId}` is not currently implemented.

+ Parameters
    + attributeId (number) - The identifier of the attribute for which to retrieve sample information
        + See /networkDevices/{id}/attributes for possible values
    + startTime (string) - The ISO-8601 encoded dateTime representing the earliest sample to return
    + endTime (string) - The ISO-8601 encoded dateTime representing the latest sample to return
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `1000`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `timestamp`
        + Default: `timestamp`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(NetworkDeviceValueSamplesEntity)

### Get Alarms For A Network Device [GET /networkDevices/{id}/alarms{?startTime,endTime,priorityMinimum,priorityMaximum,type,excludePending,excludeAcknowledged,excludeDiscarded,attribute,page,pageSize,sort}]
Retrieves a collection of alarms for the specified network device.

+ Parameters
    + startTime (string, optional) - The ISO-8601 encoded dateTime representing the earliest alarm to return
    + endTime (string, optional) - The ISO-8601 encoded dateTime representing the latest alarm to return
    + priorityMinimum (number, optional) - The minimum priority of the requested alarms
    + priorityMaximum (number, optional) - The maximum priority of the requested alarms
    + type (number, optional) - The type of the requested alarms
        + See /enumSets/505/members for possible values.
    + excludePending (boolean, optional) - The flag to exclude pending alarms
        + Default: `false`
    + excludeAcknowledged (boolean, optional) - The flag to exclude acknowledged alarms
        + Default: `false`
    + excludeDiscarded (boolean, optional) - The flag to exclude discarded alarms
        + Default: `false`
    + attribute (number, optional) - The attribute of the requested alarms
        + See /enumSets/509/members for possible values
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `priority`, `creationTime`
        + Default: `creationTime`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes(NetworkDeviceAlarmsEntity)

Group Objects
============

This section contains information about object-related resources as used within the Metasys API.

## Object Collection [/objects{?type,page,pageSize,sort}]
A collection of all objects in the system.

+ Parameters
    + type (number) - Type of object to return
        + See /enumSets/508/members for possible values
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `typeId`
        + Default: `name`

### Get Objects [GET]
Retrieves a collection of objects. Note that this endpoint requires the `type` parameter to be supplied; returning a list of all objects in the system is not currently supported.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include ObjectEntityCollection
        + self:`/objects{?queryParams}` (string) - A link to this object data set

## Object [/objects/{id}]
Represents a single item (object) in the system

+ Parameters
    + id (string) - The identifier of the object

### Get A Single Object [GET]
Retrieves the specified object.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)

    + Headers

            Allow: GET

    + Attributes(ObjectEntity)

### Get Object Parent [GET /objects/{id}/isChildOf]
Retrieves the immediate parent of the specified object.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)

    + Headers

            Allow: GET

    + Attributes(ObjectEntity)

### Get Object Children [GET /objects/{id}/hasChildren{?page,pageSize,sort}]
Retrieves the children (recursively) of the specified object.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)

    + Headers

            Allow: GET

    + Attributes
        + Include ObjectEntityCollection
        + self:`/objects/{id}/hasChildren{?queryParams}` (string) - A link to this object data set

### Get Network Device Of An Object [GET /objects/{id}/usedByNetworkDevice]
Retrieves the immediate parent network device that contains this object.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)

    + Headers

            Allow: GET

    + Attributes(NetworkDeviceEntity)

### Get Equipment Points Mapped To An Object [GET /objects/{id}/hasEquipmentMappings{?page,pageSize,sort}]
Retrieves all equipment points mapped to attributes of this object.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `category`,`equipmentName`, `isDisplayData`, `shortName`
        + Default: `shortName`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EquipmentPointMappingCollection
        + self:`/objects/{id}/hasEquipmentMappings{?queryParams}` (string) - A link to this equipment data set

### Get Object Attributes With Samples [GET /objects/{id}/attributes]
Retrieves a collection of attributes under the specified object for which samples are available.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include ObjectAttributeWithSamplesCollection
        + self:`/objects/{id}/attributes` - A link to this data set

### Get Samples For An Object Attribute [GET /objects/{id}/attributes/{attributeId}/samples{?startTime,endTime,page,pageSize,sort}]
Retrieves a collection of samples for the specified object attribute during a particular date and time range.

Note that the parent endpoint `/objects/{id}/attributes/{attributeId}` is not currently implemented.

+ Parameters
    + attributeId (number) - The identifier of the attribute for which to retrieve sample information
        + See /points/{id}/attributes for possible values
    + startTime (string) - The ISO-8601 encoded dateTime representing the earliest sample to return
    + endTime (string) - The ISO-8601 encoded dateTime representing the latest sample to return
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `1000`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `timestamp`
        + Default: `timestamp`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes (ObjectValueSamplesEntity)

### Get Alarms For An Object [GET /objects/{id}/alarms{?startTime,endTime,priorityMinimum,priorityMaximum,type,excludePending,excludeAcknowledged,excludeDiscarded,attribute,page,pageSize,sort}]
Retrieves a collection of alarms for the specified object.

+ Parameters
    + startTime (string, optional) - The ISO-8601 encoded dateTime representing the earliest alarm to return
    + endTime (string, optional) - The ISO-8601 encoded dateTime representing the latest alarm to return
    + priorityMinimum (number, optional) - The minimum priority of the requested alarms
    + priorityMaximum (number, optional) - The maximum priority of the requested alarms
    + type (number, optional) - The type of the requested alarms
        + See /enumSets/505/members for possible values.
    + excludePending (boolean, optional) - The flag to exclude pending alarms
        + Default: `false`
    + excludeAcknowledged (boolean, optional) - The flag to exclude acknowledged alarms
        + Default: `false`
    + excludeDiscarded (boolean, optional) - The flag to exclude discarded alarms
        + Default: `false`
    + attribute (number, optional) - The attribute of the requested alarms
        + See /enumSets/509/members for possible values
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `priority`, `creationTime`
        + Default: `creationTime`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes (ObjectAlarmsEntity)

### Get Activities For An Object [GET /objects/{id}/activities{?originApplications,classLevels,actions,ids,startTime,endTime,page,pageSize,sort,discarded}]
Retrieves a collection of activities for the specified object. When using multiple filters, an AND evaluation is assumed.

+ Parameters
    + originApplications:`1,2` (string, optional) - Filter by comma-separated list of origin applications
        + See /enumSets/578/members for possible values
    + classLevels:`1,2` (string, optional) - Filter by comma-separated list of class levels
        + See /enumSets/568/members for possible values
    + actions:`1,2` (string, optional) - Filter by comma-separated list of the actions
        + See /enumSets/577/members for possible values
    + ids:`E1C28F29-90DC-464F-90DE-07DC08454E31,483FB3F7-6C65-4570-9DC2-01F4FD1A8F3A` (string, optional) -  Filter by comma-separated list of identifiers of the audits
    + startTime:`2018-05-21T13:58:20.243Z` (string, optional) - The ISO-8601 encoded dateTime representing the earliest activity to return
    + endTime:`2018-05-22T13:58:20.243Z` (string, optional) - The ISO-8601 encoded dateTime representing the latest activity to return
    + discarded (boolean, optional) - Filter for activities that have been discarded. True = only discarded activities. False = only non-discarded activities.  Empty = all activities
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `creationTime`
        + Default: `-creationTime`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET
    + Attributes
        + Include ActivityEntityCollection
        + self:`/objects/{id}/activities{?queryParams}` (string) - A link to this activity data set

Group Spaces
============

This section contains information about space-related resources as used within the Metasys API.

## Space Collection [/spaces{?type,page,pageSize,sort}]
A collection of all spaces

+ Parameters
    + type (number, optional) - Type of space to return
        + See /enumSets/1766/members for possible values
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `typeId`
        + Default: `name`

### Get Spaces [GET]
Retrieves a collection of spaces.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include SpaceEntityCollection
        + self:`/spaces{?queryParams}` (string) - A link to this space data set

## Space [/spaces/{id}]
A specific space in the system

+ Parameters
    + id (string) - The identifier of the space

### Get A Single Space [GET]
Retrieves the specified space.

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes (SpaceEntity)

### Get Equipment Serving A Space [GET /spaces/{id}/servedBy{?page,pageSize,sort}]
Retrieves the collection of equipment that serve the specified space.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `type`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include EquipmentEntityCollection
        + self:`/spaces/{id}/servedBy{?queryParams}` (string) - A link to this equipment data set

### Get Space Parent [GET /spaces/{id}/isLocatedIn]
Retrieves the space that is the parent of the specified space.

+ Parameters
    + id (string) - The identifier of the equipment for which to retrieve serving equipment

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes (SpaceEntity)

### Get Space Children [GET /spaces/{id}/contains{?page,pageSize,sort}]
Retrieves the collection of spaces that are located in the specified space.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include SpaceEntityCollection
        + self:`/spaces/{id}/contains{?queryParams}` (string) - A link to this space data set

### Get Network Devices Hosted By A Space [GET /spaces/{id}/hosts{?page,pageSize,sort}]
Retrieves the collection of network devices that are hosted by the specified space. A space is considered to host a network device is considered to host an equipment if any equipment instance serving the space defines points that map to an attribute of any object contained on the network device.

+ Parameters
    + page (number, optional) - The page number of items to return
        + Default: `1`
    + pageSize (number, optional) - The maximum number of items to return in the response
        + Default: `100`
    + sort (string, optional) - The criteria to use when sorting results (see [rules](#sorting-rules))
        + Accepted Values: `categoryId`, `firmwareVersion`, `itemReference`, `name`, `typeId`
        + Default: `name`

+ Request
    + Headers

            Accept: application/vnd.metasysapi.v1+json
            Authorization: Bearer [token]

+ Response 200 (application/vnd.metasysapi.v1+json)
    + Headers

            Allow: GET

    + Attributes
        + Include NetworkDeviceEntityCollection
        + self:`/spaces/{id}/hosts{?queryParams}` (string) - A link to this network device data set

# Data Structures

## Signature
+ fullName:`John Smith` (string) - The name of the person "signing"
+ reason (string) - The reason for signing

## ItemCollection
+ total:`1` (number) - The total number of items in the collection
+ items (object) - The collection of items
+ next:`[link to next page]` (string) - A link to the next page of results
+ previous:`[link to previous page]` (string) - A link to the previous page of results

## BaseObject (object)
+ id:`00000000-0000-0000-0000-000000000000` (string) - The identifier of the object
+ itemReference:`site:device/itemReference` (string) - The fully qualified reference of the object
+ name (string) - The name of the object

## SpaceLinkCollection
+ Include ItemCollection
+ items:`[/spaces/{id}]` (array[string]) - The collection of space links

## SpaceEntity
+ Include SpaceData
+ Include SpaceRelationshipLinks

## SpaceEntityCollection
+ Include ItemCollection
+ items (array[SpaceEntity], fixed-type) - The collection of spaces

## SpaceData
+ Include BaseObject
+ name (string) - The name of the space
+ type:`/enumSets/{id}/members/{memberId}` (string) - A link to the type of space

## SpaceRelationshipLinks
+ self:`/spaces/{id}` (string) - A link to this space
+ isLocatedIn:`/spaces/{id}` (string) - A link to the space in which this space is located
+ contains:`/spaces/{id}/contains` (string) - A link to the spaces contained within this space
+ servedBy:`/spaces/{id}/servedBy` (string) - A link to the equipment instances serving this space
+ hosts:`/spaces/{id}/hosts` (string) - A link to the network devices hosted by this space

## EquipmentLinkCollection
+ Include ItemCollection
+ items:`[/equipment/{id}]` (array[string]) - The collection of equipment links

## EquipmentEntity
+ Include EquipmentData
+ Include EquipmentRelationshipLinks

## EquipmentEntityCollection
+ Include ItemCollection
+ items (array[EquipmentEntity], fixed-type) - The collection of equipment instances

## EquipmentData
+ Include BaseObject
+ name (string) - The full name of the equipment instance
+ type:`AHU` (string) - The name of the equipment definition for this equipment instance

## EquipmentRelationshipLinks
+ self:`/equipment/{id}` (string) - A link to this equipment instance
+ spacesServed:`/equipment/{id}/spacesServed` (string) - A link to the spaces served by this equipment instance
+ equipmentServed:`/equipment/{id}/equipmentServed` (string) - A link to the equipment served by this equipment instance
+ servedBy:`/equipment/{id}/servedBy` (string) - A link to the equipment that serves this equipment instance
+ hostedBy:`/equipment/{id}/hostedBy` (string) - A link to the network devices that host this equipment instance
+ hasMappings:`/equipment/{id}/hasMappings` (string) - A link to the point mappings defined by this equipment instance

## EquipmentPointMappingEntity
+ Include EquipmentPointMappingData
+ Include EquipmentPointMappingRelationshipLinks

## EquipmentPointMappingCollection
+ Include ItemCollection
+ items (array[EquipmentPointMappingEntity], fixed-type) - The collection of equipment point mappings

## EquipmentPointMappingData
+ equipmentName (string) - The name of the mapped equipment
+ shortName:`ZN-T` (string) - The short name for the equipment point
+ label:`Zone Temperature` (string) - The label for the equipment point
+ category (string) - The category for the equipment point
+ isDisplayData (boolean) - Whether the equipment point is marked as display data

## EquipmentPointMappingRelationshipLinks
+ mappedEquipment:`/equipment/{id}` (string) - A link to the mapped equipment
+ mappedObject:`/objects/{id}` (string) - A link to the mapped object
+ mappedAttribute:`/enumSets/509/members/{memberId}` (string) - A link to the mapped attribute

## NetworkDeviceLinkCollection
+ Include ItemCollection
+ items:`/networkDevices/{id}` (array[string]) - The collection of network device links

## NetworkDeviceEntity
+ Include NetworkDeviceData
+ Include NetworkDeviceRelationshipLinks

## NetworkDeviceEntityCollection
+ Include ItemCollection
+ items (array[NetworkDeviceEntity], fixed-type) - The collection of network devices

## NetworkDeviceData
+ Include BaseObject
+ name (string) - The name of the network device
+ type:`/enumSets/{id}/members/{memberId}` (string) - A link to the type of network device
+ description (string) - The description for the network device
+ firmwareVersion:`10.0.0` (string) - The firmware version for the network device
+ category:`/enumSets/{id}/members/{memberId}` (string) - A link to the authorization category for the network device
+ timeZone:`/enumSets/{id}/members/{memberId}` (string) - A link to the time zone in which the device is located

## NetworkDeviceRelationshipLinks
+ self:`/networkDevices/{id}` (string) - The link to the network device
+ isChildOf:`/networkDevices/{id}` (string) - A link to the network device that is the parent of this network device
+ hasChildren:`/networkDevices/{id}/hasChildren` (string) - A link to the network devices that are children of this network device
+ hosts:`/networkDevices/{id}/hosts` (string) - A link to the equipment instances hosted by this network device
+ hostedBy:`/networkDevices/{id}/hostedBy` (string) - A link to the spaces that host this network device
+ uses:`/networkDevices/{id}/uses` (string) - A link to the objects contained within this network device
+ attributes:`/networkDevices/{id}/attributes` (string) - A link to the attributes on this network device for which samples are available
+ alarms:`/networkDevices/{id}/alarms` (string) - A link to the alarms available for this network device

## ObjectLinkCollection
+ Include ItemCollection
+ items:`[/objects/{id}]` (array[string]) - The collection of object links

## ObjectEntity
+ Include ObjectData
+ Include ObjectRelationshipLinks

## ObjectEntityCollection
+ Include ItemCollection
+ items (array[ObjectEntity], fixed-type) - The collection of objects

## ObjectData
+ Include BaseObject
+ name (string) - The name of the object
+ type:`enumSets/{id}/members/{memberId}` (string) - A link to the type of object

## ObjectRelationshipLinks
+ self (string) - A link to this object
+ isChildOf:`/objects/{id}/isChildOf` (string) - A link to the parent of this object
+ hasChildren:`/objects/{id}/hasChildren` (string) - A link to the children of this object
+ usedByNetworkDevice:`/objects/{id}/usedByNetworkDevice` (string) - A link to the network device that contains this object
+ hasEquipmentMappings:`/objects/{id}/hasEquipmentMappings` (string) - A link to all equipment point mappings defined for this object
+ attributes:`/objects/{id}/attributes` (string) - A link to the attributes on this object for which samples are available
+ alarms:`/objects/{id}/alarms` (string) - A link to the alarms generated by this object

## ObjectAttributeWithSamples
+ samples:`/objects/{id}/attributes/{attributeId}/samples` (string) - A link to this collection of object value samples
+ attribute:`enumSets/{id}/members/{memberId}` (string) - A link to the attribute on which the samples were taken

## NetworkDeviceAttributeWithSamples
+ samples:`/networkDevice/{id}/attributes/{attributeId}/samples` (string) - A link to the collection of network device value samples
+ attribute:`enumSets/{id}/members/{memberId}` (string) - A link to the attribute on which the samples were taken

## ObjectValueSamplesEntity
+ Include ValueSamplesData
+ Include ObjectValueSamplesRelationshipLinks

## NetworkDeviceValueSamplesEntity
+ Include ValueSamplesData
+ Include NetworkDeviceValueSamplesRelationshipLinks

## ObjectAttributeWithSamplesCollection
+ Include ItemCollection
+ items (array[ObjectAttributeWithSamples], fixed-type) - The collection of available samples

## NetworkDeviceAttributeWithSamplesCollection
+ Include ItemCollection
+ items (array[NetworkDeviceAttributeWithSamples], fixed-type) - The collection of available samples

## ValueSamplesData
+ Include ItemCollection
+ items (array[ValueSample], fixed-type) - The collection of samples
+ attribute:`/enumSets/509/members/{memberId}` - A link to the attribute for which the samples were generated

## ObjectValueSamplesRelationshipLinks
+ self:`/objects/{id}/attributes/{attributeId}/samples` (string) - A link to this collection of object value samples
+ object:`/objects/{id}` (string) - A link to the object from which these samples were generated

## NetworkDeviceValueSamplesRelationshipLinks
+ self:`/networkDevice/{id}/attributes/{attributeId}/samples` (string) - A link to this collection of network device value samples
+ networkDevice:`/networkDevices/{id}` (string) - A link to the network device from which these samples were generated

## ValueSample
+ One Of
    + value (AnalogValueSampleRaw) - The sample value
    + value (DigitalValuesSampleRaw) - The sample value
+ timestamp:`2000-01-01T12:00:00Z` (string) - The ISO-8601 encoded timestamp for when the sample was recorded
+ isReliable (boolean) - The flag indicating whether the source of the sample was in a reliable state when the sample was recorded

## AnalogValueSampleRaw
+ value:`68.9` (number) - The value of the sample
+ units:`/enumSets/{id}/members/{memberId}` (string) - A link to the units of the sample

## DigitalValuesSampleRaw
+ value:`1` (number) - The value of the sample
+ valueEnumMember:`/enumSets/{id}/members/{memberId}` (string) - A link to the enum member for the sample value

## NetworkDeviceAlarmsRelationshipLinks
+ self:`/networkDevices/{id}/alarms` (string) - A link to this collection of network device alarms
+ networkDevice:`/networkDevices/{id}` (string) - A link to the network device from which this alarm was generated

## NetworkDeviceAlarmRelationshipLinks
+ self:`/networkDevices/{id}/alarms/{alarmId}` (string) - A link to this network device alarm
+ networkDevice:`/networkDevices/{id}` (string) - A link to the network device from which this alarm was generated

## NetworkDeviceAlarmEntity
+ Include AlarmEntity
+ Include NetworkDeviceAlarmRelationshipLinks

## NetworkDeviceAlarmsEntity
+ Include AlarmEntityCollection
+ Include NetworkDeviceAlarmsRelationshipLinks

## NetworkDeviceAnnotationsRelationshipLinks
+ self:`/networkDevices/{id}/alarms/{alarmId}/annotations` (string) - A link to this collection of network device alarm annotations
+ networkDeviceAlarm:`/networkDevices/{id}/alarms/{alarmId}` (string) - A link to the alarm on the network device on which these annotations were created on

## NetworkDeviceAlarmAnnotationsEntity
+ Include AlarmAnnotationEntityCollection
+ Include NetworkDeviceAnnotationsRelationshipLinks

## ObjectAlarmsRelationshipLinks
+ self:`/objects/{id}/alarms` (string) - A link to this collection of object alarms
+ object:`/objects/{id}` (string) - A link to the object from which which this alarm was generated

## ObjectAlarmRelationshipLinks
+ self:`/objects/{id}/alarms/{alarmId}` (string) - A link to this object alarm
+ object:`/objects/{id}` (string) - A link to the object from which this alarm was generated

## ObjectAlarmEntity
+ Include AlarmEntity
+ Include ObjectAlarmRelationshipLinks

## ObjectAlarmsEntity
+ Include AlarmEntityCollection
+ Include ObjectAlarmsRelationshipLinks

## ObjectAnnotationsRelationshipLinks
+ self:`/objects/{id}/alarms/{alarmId}/annotations` (string) - A link to this collection of object alarm annotations
+ objectAlarm:`/objects/{id}/alarms/{alarmId}` (string) - A link to the alarm on the object on which these annotations were created on

## ObjectAlarmAnnotationsEntity
+ Include ObjectEntityCollection
+ Include ObjectAnnotationsRelationshipLinks

## AlarmEntity
+ Include BaseObject
+ message (string) - The alarm message
+ isAckRequired (boolean) - Indicates whether an acknowledgment is required on the alarm
+ type:`/enumSets/{id}/members/{memberId}` (string) - A link to the enum member for the type of alarm
+ priority (number) - The priority of the alarm
+ One Of
    + triggerValue (AnalogValueAlarm) - The value of the attribute of the object on which the alarm was triggered
    + triggerValue (DigitalValueAlarm) - The value of the attribute of the object on which the alarm was triggered
+ creationTime:`2000-01-01T12:00:00Z` (string) - The ISO-8601 encoded timestamp for when the alarm was detected
+ isAcknowledged (boolean) - Indicates whether the alarm has been acknowledged
+ isDiscarded (boolean) - Indicates whether the alarm has been discarded
+ category:`/enumSets/{id}/members/{memberId}` (string) - A link to the authorization category of the object when the alarm was generated
+ object:`/objects/{id}` (string) - A link to the object from which this alarm was generated
+ attribute:`/enumSets/509/members/{memberId}` (string) - A link to the attribute for which this alarm was generated
+ annotations:`/alarms/{id}/annotations` (string) - A link to the annotations available on the alarm
+ self:`/alarms/{id}` (string) - A link to this collection of alarms

## AlarmEntityCollection
+ Include ItemCollection
+ items (array[AlarmEntity], fixed-type) - The collection of alarms

## AlarmAnnotationEntityCollection
+ Include ItemCollection
+ items (array[AlarmAnnotationEntity], fixed-type) - The collection of annotations for an alarm

## AlarmAnnotationEntity
+ text (string) - The annotation text
+ user (string) - The user who annotated the alarm
+ creationTime:`2000-01-01T12:00:00Z` (string) - The ISO-8601 encoded timestamp for when the annotation on the alarm was created
+ action (AlarmActionEnum): The enum for the alarm action on which the annotation was created. In case of the annotation was not created during an alarm action, the set of the enum will be zero.
+ alarm:`/alarms/{id}` (string)  - A link to the alarm

## AnalogValueAlarm
+ value:`68.9` (number) - The value of the attribute of the object on which the alarm was triggered
+ units:`/enumSets/{id}/members/{memberId}` (string) - A link to the units of the object on which the alarm was triggered

## DigitalValueAlarm
+ value:`1` (number) - The value of the attribute of the object on which the alarm was triggered
+ valueEnumMember:`/enumSets/{id}/members/{memberId}` (string) - A link to the enum member for the alarm trigger value

## ActivityEntityCollection
+ Include ItemCollection
+ items (array[ActivityEntity], fixed-type) - The collection of alarms

## ActivityEntity
+ id:`00000000-0000-0000-0000-000000000000` (string) - The identifier of the activity
+ productName (JciProductName) - The Product enum value of this activity message
+ creationTime:`2017-09-18T08:11:28Z` (string) - The ISO-8601 encoded dateTime representing the creation time when this activity message was created
+ actionType (ProductEnumeration) - See ProductEnumeration
    + See /enumSets/577/members for possible values
+ discarded (boolean) - Indicates if the activity has been discarded
+ status (ProductEnumeration) - Enumeration representing status
    + See /enumSets/516/members for possible values
+ preData (Data) - Data value prior to the Activity
+ postData (Data) - Data value after the Activity
+ parameters (array[Data], fixed-type) The actual data that is changing
+ errorString (string) - The error that may have occurred during an activity
+ user (string) - The userName of the user that initiated the activity
+ signature (Signature) - The user who created this activity
+ object:`/objects/{id}` (string) - A link to the object on which the activity was generated
+ self:`/activities/{id}` (string) - A link to this activity
+ annotations:`/activities/{id}/annotations` (string) - Link to annotations
+ legacy (LegacyMetasys) - Metasys specific data

## ProductEnumeration
+ enumSetId (number) - The enumeration set identifier
+ enumMemberId (number) - The enumeration member

## Data
+ value (string) - The data value
+ type (string) -  The data type
+ additionalData:`"unit":"2", "precision":"1"` (string) - Key value pairs of any additional data

## ActivityMessageEditMethod (enum)
+ `0` - Unknown
+ `1` - Manual
+ `2` - Automated
+ `3` - Imported
+ `4` - GenericPersonAsOperator

## ObjectDefinition
+ primaryObjectId (string) - the primary identifier
+ primaryObjectType (string) - the object type value
+ primaryObjectOriginatingProduct (JciProductName) - the enum value of the product

## User
+ name (string) - The user name
+ identifier (string) - The identifier for the user

## Legacy
+ productName (JciProductName) - The name of the JCI product

## LegacyMetasys
+ Include Legacy
+ fullyQualifiedItemReference (string) - The fully qualified item reference of the object that created this activity
+ itemName (string) - The short name of the object that created this activity
+ classLevel (ProductEnumeration) - Indicates the class level in which this activity belongs
    + See /enumSets/568/members for possible values
+ originApplication (ProductEnumeration) - Indicates the application that performed action that generated this activity
    + See /enumSets/578/members for possible values
+ description (ProductEnumeration) - Provides the description of the action that generated this activity
    + See /enumSets/580/members for possible values

## JciProductName (enum)
+ '0' - Metasys
+ '1' - CCURE
+ '2' - Victor

## EnumSet
+ Include EnumSetData
+ Include EnumSetRelationships

## EnumSetCollection
+ Include ItemCollection
+ items (array[EnumSet], fixed-type) - The collection of enum sets

## EnumSetData
+ id:`508` (number) - The identifier for the enum set
+ description:`Object Type` (string) - The description of the enum set

## EnumSetRelationships
+ self:`enumSets/{id}` (string) - A link to the enum set
+ members:`enumSets/{id}/members` (string) - A link to the enum set members

## EnumMember
+ Include EnumMemberData
+ Include EnumMemberRelationships

## EnumMemberCollection
+ Include ItemCollection
+ items (array[EnumMember], fixed-type) - The collection of enum members

## EnumMemberData
+ id:`165` (number) - The identifier for the enum member
+ description:`JCI AV` (string) - The description of the enum member

## EnumMemberRelationships
+ self:`/enumSets/{id}/members/{memberId}` (string) - A link to the enum set members
+ set:`/enumSets/{id}` (string) - A link to the enum set

## AttributeEnum
+ Include EnumMember
+ id:`85` (number) - Identifier for the attribute
+ description:`Present Value` (string) - The description of the attribute
+ self:`/enumSets/509/members/85` (string)
+ set:`/enumSets/509` (string, fixed) - A link to the attribute enum set

## ObjectTypeEnum
+ Include EnumMember
+ id:`165` (number) - Identifier for the object type
+ description:`JCI AV` (string) - The description of the object type
+ self:`/enumSets/508/members/165` (string) - A link to the object type enum member
+ set:`/enumSets/508` (string, fixed) - A link to the object type enum set

## ObjectCategoryEnum
+ Include EnumMember
+ id:`5` (number) - Identifier for the object category
+ description:`General` (string) - The description of the object category
+ self:`/enumSets/33/members/5` (string) - A link to the object category enum member
+ set:`/enumSets/33` (string, fixed) - Link to the object category enum set

## SpaceTypeEnum
+ Include EnumMember
+ id:`1` (number) - Identifier for the space type
+ description:`Building` (string) - The description of the space type
+ self:`/enumSets/1766/members/1` (string) - A link to the space type enum member
+ set:`/enumSets/1766` (string, fixed) - Link to the space type enum set

## TimeZoneEnum
+ Include EnumMember
+ id:`53` (number) - Identifier for the time zone
+ description:`(UTC-06:00) Central Time (US & Canada)` (string) - The description of the time zone, consistent with SMP display
+ self:`/enumSets/576/members/53`(string)  - A link to the time zone enum member
+ set:`/enumSets/576` (string, fixed) - Link to the time zone enum set

## UnitsEnum
+ Include EnumMember
+ id:`64` (number) - The identifier for the unit
+ description:`deg F` (string) - The description of the unit
+ self:`/enumSets/507/members/64` (string)  - A link to the units enum member
+ set:`/enumSets/507` (string, fixed) - A link to the units enum set

## ObjectStatusEnum
+ Include EnumMember
+ id:`38` (number) - The identifier for the status
+ description:`Alarm` (string) - The description of the status
+ self:`/enumSets/505/members/38` (string) - A link to the object status enum member
+ set:`/enumSets/505` (string, fixed) - A link to the object status enum set

## AlarmActionEnum
+ Include EnumMember
+ id:`64` (number) - The identifier for the action
+ description:`deg F` (string) - The description of the action
+ self:`/enumSets/xxxx/members/1` - A link to the alarm action enum member
+ set:`/enumSets/xxxx` (fixed) - A link to the alarm action enum set

## TokenRequestBase
+ `client_id`: `metasys_ui` (string, required) - Accepted Values: `metasys_ui`. The client's identifier with respect to the authentication service.
+ client_secret: `secret` (string, required) - Accepted Values: `secret`. The secret key for client authentication, a placeholder.
+ scope: `metasys_api openid offline_access` (string, required) - Accepted Values: `metasys_api`, `openid`, `offline_access`. The `openid` value that specifies behavior from what is being requested and how the token will be used. `metasys_api` defines the scope of the resource the user is asking for; `openid` is required by OpenIDConnect; `offline_access` allows for retrieving a refresh token for sliding expiration.
+ `grant_type`: `password` (string, required) - Accepted Values: `password`, `refresh_token`. Use `password` for initial login. Use `refresh_token` to refresh access token.

## TokenRequestLogin
+ Include TokenRequestBase
+ username (string, required) - {Metasys Username} 
+ password (string, required) - {Metasys Password}

## TokenRequestRefresh
+ Include TokenRequestBase
+ `refresh_token` (string, required) - The refresh token that was received at login.

## TokenResponse
+ access_token (string) - The access token which must be included in an Authorization header on all calls requiring authentication. 
+ expires_in (number) - Seconds to Expiration
+ token_type (string) - Indicates the type of token. This will always be *Bearer*
+ `refresh_token` (string) - The refresh token is used to extend a session. A refresh token is only returned if `offline_access` is included in `scope` when making requests of `/Authentication/LogIn`.

## RefreshTokenResponse
+ Include TokenResponse
+ id_token (string) - Identity Token
