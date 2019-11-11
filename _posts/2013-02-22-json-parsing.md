---
layout: post
title:  "JSON Parsing"
date:   2013-02-22 15:05:00 -0600
categories: java json
---

Recently, I had a requirement, where in I had to read a property file which contains JSON strings.
There were some fields, which were required for internal processing, and non client facing.

### Application Details:
The application was a spring application, with RESTFul services, and standalone data workers.

The RESTFul services exposed the JSON string to the clients (mobile applications and node.js applications).

Coming to the actual problem, the JSON string represented the following object, and I was required to hide the companyName, startDate from the client. Also, I required those fields for internal processing.

```
public class Employee implements Serializable {

  private static final long serialVersionUID = 2795458242408588641L;
  private String companyName;
  private String employeeName;
  private String startDate;

  @JsonIgnore
  public String getCompanyName() {
    return companyName;
  }

  public void setCompanyName(String companyName) {
    this.companyName = companyName;
  }

  public String getEmployeeName() {
    return employeeName;
  }

  public void setEmployeeName(String employeeName) {
    this.employeeName = employeeName;
  }

  @JsonIgnore
  public String getStartDate() {
    return startDate;
  }

  public void setStartDate(String startDate) {
    this.startDate = startDate;
  }
}
```

Notice the `@JsonIgnore` annotation on the methods getCompanyName() and getStartDate().
When the Jackson parser, parses the above object into a JSON string, the fields companyName and startDate are ignored from the output.

But now the problem was Jackson parser was ignore the fields companyName and startDate, when I was reading the json string from the properties file.

The fix was simple:
```
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.disable(Feature.USE_ANNOTATIONS);
```

Feature - `org.codehaus.jackson.map.DeserializationConfig.Feature`

If there is a need to exclude all null fields from the json, before its sent over the wire, set the following parameter:

```
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(Inclusion.NON_NULL);
```

After setting the above attribute, the code to read the json string from properties file works fine.
Also the fields are not exposed to the end user from the controllers.

Hope this was helpful!!
