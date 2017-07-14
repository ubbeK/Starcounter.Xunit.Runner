# Starcounter2.3_xunit
Proof of concept for testing Starcounter Apps and Databases in runtime. Import `ScXunitRunner` to a normal Starcounter version 2.3.1.X App and start write the tests.

## ScXunitRunner
Using:
* .NET Framework 4.5.2
* Starcounter version 2.3.1.X. 
* xunit 2.2.0
* xunit.runner.console 2.2.0

Adds a Starcounter handler, `/ScXunitRunner/<CallingAssemblyName>`, which executes `Xunit` tests as a xunit console application in the same AppDomain as the Starcounter database.

## ScTestApp
A Starcounter version 2.3.1.X test App which imports `ScXunitRunner`. It needs to create an instance of ScXunitRunner in order to generate test Handler and for the tests to be found.
```c#
public class Program
{
    static void Main()
    {
        StarcounterXunitRunner runner = new StarcounterXunitRunner();
    }
}
```

This project contains a bunch of examples to illustrate all the features `ScXunitRunner`. It also creates `DummyApp.DummyAppDb` rows for verifying the proof of concept.

TestCase example:
```c#
[Fact]
public void TestCase_AddingNewRow()
{
    Scheduling.ScheduleTask(() => {
        int entriesBeforeCount = Db.SQL<DummyAppDb>("SELECT x FROM DummyApp.DummyAppDb x").Count();

        Db.Transact(() =>
        {
            DummyAppDb entry = new DummyAppDb();
        });

        int entriesAfterCount = Db.SQL<DummyAppDb>("SELECT x FROM DummyApp.DummyAppDb x").Count();

        Assert.True(int.Equals(entriesBeforeCount + 1, entriesAfterCount), $"DummyApp.DummyAppDb beforeCount={entriesBeforeCount} is not one less than afterCount={entriesAfterCount}");
    }, waitForCompletion: true);
}
```

Test result from running `/ScXunitRunner/ScTestApp` handler.
```
[1/9] ScTestApp.TestSetDoNotSaveContext.TestSetDoNotSaveContext_Test.................. PASSED. Execution time: 1,039s
[2/9] ScTestApp.TestSetDoNotSaveContext.TestSetDoNotSaveContext_Test2................. PASSED. Execution time: 0,000s
[3/9] ScTestApp.TestSetDoNotSaveContext.TestSetDoNotSaveContext_Test3................. PASSED. Execution time: 0,000s
[4/9] ScTestApp.TestSetAlwaysFailing.TestCase_AlwaysFailing_1......................... FAILED. Execution time: 2,007s
This assertion will always fail
Expected: True
Actual:   False   at Xunit.Assert.True(Nullable`1 condition, String userMessage)
   at ScTestApp.TestSetAlwaysFailing.TestCase_AlwaysFailing_1() in C:\GitHub\home\Starcounter2.3_xunit\ScTestApp\TestSetAlwaysFailing.cs:line 13
[5/9] ScTestApp.TestSetAddingDatabaseEntries.TestCase_AddingNewRow_3.................. PASSED. Execution time: 2,014s
[6/9] ScTestApp.TestSetAlwaysFailing.TestCase_AlwaysFailing_2......................... FAILED. Execution time: 0,531s
This assertion will always fail
Expected: True
Actual:   False   at Xunit.Assert.True(Nullable`1 condition, String userMessage)
   at ScTestApp.TestSetAlwaysFailing.TestCase_AlwaysFailing_2() in C:\GitHub\home\Starcounter2.3_xunit\ScTestApp\TestSetAlwaysFailing.cs:line 19
[7/9] ScTestApp.TestSetAddingDatabaseEntries.TestCase_AddingNewRow_1.................. PASSED. Execution time: 0,560s
[8/9] ScTestApp.TestSetAlwaysFailing.TestCase_AlwaysFailing_3......................... FAILED. Execution time: 0,526s
This assertion will always fail
Expected: True
Actual:   False   at Xunit.Assert.True(Nullable`1 condition, String userMessage)
   at ScTestApp.TestSetAlwaysFailing.TestCase_AlwaysFailing_3() in C:\GitHub\home\Starcounter2.3_xunit\ScTestApp\TestSetAlwaysFailing.cs:line 25
[9/9] ScTestApp.TestSetAddingDatabaseEntries.TestCase_AddingNewRow_2.................. PASSED. Execution time: 0,501s

Total execution time: 3,158s, Passed: 6, Failed: 3, Skipped: 0
```

## DummyApp
A dummy Starcounter version 2.3.1.X App which only has one database-table. Used for testing purposes only.
```c#
[Database]
public class DummyAppDb
{
    public string Name { get; set; }
    public int Integer { get; set; }
    public DateTime DateCreated { get; set; }
}
```

It has handlers for the following
* `/DummyApp/createnewentry` - create a new row with random data
* `/DummyApp/createnewentry/<# of new entries>` - create multiple new rows with random data
* `/DummyApp/deleteall` - delete all rows
* `/DummyApp/listentries` - list all rows in text format