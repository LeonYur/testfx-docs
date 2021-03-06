# RFC 001- Framework Extensibility for Description Attributes

## Summary
This details the MSTest V2 framework extensibility for attributes that describe a test.  

## Motivation
It is a requirement for teams to have description attributes which are strongly typed over a test method as opposed to just a KeyvaluePair<string,string>. This allows teams to have a standard across the team which is less error prone and more natural to specify. Users can also filter tests based on the values of these attributes.

## Detailed Design

### Requirements
1. One should be able to have a strongly typed desciption attribute on a test method.
2. One should be able to filter in VS IDE based on this description attribute.
3. One should be able to see this information in Test Reporting.
4. This extensibility should also guarantee that attributes in MSTest V1 which are brought into MSTest V2 with this extensibility model should be source compatible.

### Proposed solution
The test framework currently has a TestProperty attribute which can be used to define custom traits as a KeyValuePair<string,string>. The definition of this attribute is as below:
```csharp
    [AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
    public sealed class TestPropertyAttribute : Attribute
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="TestPropertyAttribute"/> class.
        /// </summary>
        /// <param name="name">
        /// The name.
        /// </param>
        /// <param name="value">
        /// The value.
        /// </param>
        public TestPropertyAttribute(string name, string value)
        {
            this.Name = name;
            this.Value = value;
        }

        /// <summary>
        /// Gets the name.
        /// </summary>
        public string Name { get; }

        /// <summary>
        /// Gets the value.
        /// </summary>
        public string Value { get; }
    }
``` 
Here is how the test methods are decorated with this attribute:
```csharp
        [TestMethod]
        [TestProperty("WorkItem","234")]
        public void TestMethod()
        {
        }
```
Users can then filter tests in VS IDE Test Explorer based on this Test Property. The query string that would filter the test above would look like 
```
Trait:"WorkItem" Trait:"234"
```
This TestProperty is also filled into the TestPlatform's TestCase object which makes it available for reporting in the various loggers that can be plugged into the TestPlatform. 

To provide extension writers with the ability to have strongly typed attributes to achieve what TestProperty above achieves, the proposal is to make TestProperty a non-sealed class allowing classes to extend it like below:
```csharp
    [AttributeUsage(AttributeTargets.Method, AllowMultiple = true)]
    public class WorkItemAttribute : TestPropertyAttribute
    {
        /// <summary>
        /// Initializes a new instance of the <see cref="WorkItemAttribute"/> class.
        /// </summary>
        /// <param name="workItemId">
        /// The Id of the workitem.
        /// </param>
        public WorkItemAttribute(int workItemId)
            : base ("WorkItem", workItemId.ToString())
        {
        }
    }
```
And test methods would be decorated in a much more convenient form as below:
```csharp
        [TestMethod]
        [WorkItem(234)]
        public void TestMethod()
        {
        }
```
This would provide extension writers with the same level of functionality that TestProperty already has with filtering and reporting plus the added ease of having a strongly typed description.  

### Requirements from Test Platform
1. The attributes that were already part of MSTest V1(Description, WorkItem, CssIteration and CssProjectStructure) should be part of the trx logger output. Currently only owner and priority are part of the trx log.
2. Custom attributes should also show up in the trx logger. This is a bigger change since it might require changes in the trx schema.

## Open Questions
1. Should this attribute be opened up to be at a class and assembly level like TestCategory Attribute?
2. Is filtering needed at the console level for these extension attributes? Currently the TestCaseFilter switch at the console level only supports - TestCategory, Priority, FullyQualifiedName, Name, ClassName. This is a finite list. In order to support filtering for extended attributes this list needs to be made dynamic.   
