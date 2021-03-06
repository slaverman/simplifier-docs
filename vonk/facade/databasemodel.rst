Mapping the database
====================

In this step you will start mapping the existing database model to FHIR resources.

Reverse engineer the database model
-----------------------------------

To use EF Core, install the package for the database provider(s) you want to target. This walkthrough uses SQL Server. For a list of
available providers see Database Providers.

* Tools > NuGet Package Manager > Package Manager Console
* Run ``Install-Package Microsoft.EntityFrameworkCore.SqlServer``

We will be using some Entity Framework Tools to create a model from the database. So we will install the tools package as well:

* Run ``Install-Package Microsoft.EntityFrameworkCore.Tools``

Now it's time to create the EF model based on your existing database.

* Tools –> NuGet Package Manager –> Package Manager Console
* Run the following command to create a model from the existing database. Adjust the Data source to your instance of SQL Server. If you receive an error stating The term 'Scaffold-DbContext' is not recognized as the name of a cmdlet, then close and reopen Visual Studio.::

    Scaffold-DbContext "MultipleActiveResultSets=true;Integrated Security=SSPI;Persist Security Info=False;Initial Catalog=ViSi;Data Source=localhost" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models

    //For localdb: Scaffold-DbContext "Server=(localdb)\mssqllocaldb;Database=ViSi;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models
    //For SQLEXPRESS: Scaffold-DBContext "Data Source=(local)\SQLEXPRESS;Initial Catalog=ViSi;Integrated Security=True" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models


The reverse engineer process created entity classes (Patient.cs & BloodPressure.cs) and a derived context (ViSiContext.cs) based on the schema of the existing database.

The entity classes are simple C# objects that represent the data you will be querying and saving. Later on you will use these classes to define your queries on and to map the resources from.

* To avoid naming confusion with the FHIR Resourcetype Patient, rename both files and classes:

  * Patient => ViSiPatient
  * BloodPressure => ViSiBloodPressure

Create your first mapping
-------------------------

#. Add a project folder Repository.
#. Add a new class ``ResourceMapper``
#. Add usings for ``Vonk.Core.Common`` and ``Vonk.Facade.Starter.Models``
#. Add a method to the class ``public IResource MapPatient(ViSiPatient source)``
#. In this method, put code to create a FHIR Patient object, and fill it's elements with data from the ViSiPatient.
#. Then return the created Patient object as an IResource (you can use the extension method ``AsIResource``).

.. attention::

    ``IResource`` is an abstraction from actual Resource objects as they are known to specific versions of the Hl7.Fhir.Net API.
    Currently the only implementation is PocoResource, but this area is likely to change in the future to support multiple versions of FHIR and possibly resources that are not valid.

