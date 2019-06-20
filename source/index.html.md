--- 

title: Patient Appointment API 

language_tabs: 
   - shell 

toc_footers: 
   - <a href='#'>Sign Up for a Developer Key</a> 
   - <a href='https://github.com/lavkumarv'>Documentation Powered by lav</a> 

includes: 
   - errors 

search: true 

--- 

# Introduction 

REST Web Service to provide access to Patient Appointment data.  

# /API/APPOINTMENT/ARRIVE/{SYSTEM}/{SOURCEID}
## ***GET*** 

**Summary:** Arrive Appointment

**Description:** Arrive an Appointment via System Code and Source System Id.

### HTTP Request 
`***GET*** /api/appointment/arrive/{system}/{sourceId}` 

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| sourceId | path | sourceId | Yes | string |
| system | path | system | Yes | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | OK |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |

# /API/APPOINTMENT/CANCEL/{SYSTEM}/{SOURCEID}
## ***GET*** 

**Summary:** Cancel Appointment

**Description:** Cancel Appointment via System Code and Source System Id.

### HTTP Request 
`***GET*** /api/appointment/cancel/{system}/{sourceId}` 

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| sourceId | path | sourceId | Yes | string |
| system | path | system | Yes | string |
| cancelCode | query | cancelCode | No | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | OK |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |

# /API/APPOINTMENT/CREATE
## ***POST*** 

**Summary:** Create Appointment

**Description:** Create Appointment.

### HTTP Request 
`***POST*** /api/appointment/create` 

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| appointmentMaintain | body | appointmentMaintain | Yes |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | OK |
| 201 | Created |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |

# /API/APPOINTMENT/NOSHOW/{SYSTEM}/{SOURCEID}
## ***GET*** 

**Summary:** NoShow an Appointment

**Description:** NoShow Appointment via System Code and Source System Id.

### HTTP Request 
`***GET*** /api/appointment/noshow/{system}/{sourceId}` 

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| sourceId | path | sourceId | Yes | string |
| system | path | system | Yes | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | OK |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |

# /API/APPOINTMENT/SEARCH
## ***POST*** 

**Summary:** Get Appointments for patient by MRN or Digital Home Id.

**Description:** Get Appointments for patient by MRN or Digital Home Id.

### HTTP Request 
`***POST*** /api/appointment/search` 

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| appointmentSearchs | body | appointmentSearchs | Yes |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | OK |
| 201 | Created |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |

# /API/APPOINTMENT/TEST
## ***GET*** 

**Summary:** Load test url

**Description:** Load test url

### HTTP Request 
`***GET*** /api/appointment/test` 

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | OK |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |

<!-- Converted with the swagger-to-slate https://github.com/lavkumarv/swagger-to-slate -->
