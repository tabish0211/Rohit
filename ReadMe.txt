add-migration InitailModel -force                  --------To Override Existing Migration
---to generate database
update-database


There should be ID column in Model


Navigation Property

Customer{

MembershiptType
//add foreign key to minimise the load
public byte MembershipTypeId{get;set;}
}


//DataBase Seeding--making sync dev/test/prod with same data 

step-1 add-migration populateMigration
step-2 in Up() method call Sql("insert query ")--here
step-3 back to package manager console type 
		update-database

//Overriding conventions

Name--nvarchar(max),null---String Name
now to override above conventions do below steps

1.One way is use dataannotation
 like [Required]
      [StringLength(60)]
	  public string Name{get;set;}

2.Fluent API--


//quering Objects

step-1 Declare with DB context
private ApplicationDbContext _context

step-2ctor---CustomerControlle(){_context=new ApplicationDbConetxt()}
step-3 oveeride Dispose(bool disposing){

_context.Dispose(disposing)

}

step-4 var customers=_context.CUstomers;//Deffered Execution i.e its not executed here will excute in iteration time

var customers=_context.CUstomers.ToList();//Immediate Execution

if cutomer ==null then return HttpNotFound()

//EagerLoading---
EntityFramework by default loads Customer Object but not MembershipType which is a part of customer
_context.Customers.Include(c=>c.MembershipType).ToList()

Please add below namespace before writing just above line of code
using System.Data.Entity;


//Package Manager ShortCut
step-1 tools>option>keyboard
step-2 type packagemanagerconsole

step-press alt/.
step-4 press ok


//Code First With New Database
DbContext--an abstarction over the database that hides every complexity like mking conncetions
,executing commands managing transactions

step-1 //Install Entity Framework

install-package EntityFramework -version:6.1.3

step-2 Create Model classes
step-3 create COntext class present using sytem.data.entity;

publcic class PlutoCOntext:DbConetxt
{
//DbSet is a table
public DbSet<Customer> customers{get;set;}
public DbSet<Authors> customers{get;set;}
public DbSet<Tags> customers{get;set;}

Public PlutoCOntext():base("name=constr"){}

}

step-4 do constr activity in web.config---providerName= system.data.sqlclient
step-5 go to PM 
enable-migrations------only one time in alifecycle of projetc
add-migartion InitialModel
update-database


//Code First with Existing Database

step-1 We hvae database genrate Domain from that
add new item>Ado.net Entity Data Model>Give Model Name

step-2 choose code First From Database
step-3 specify connectionString
step-4 bring all object except migrationhistory
step-5 make Plural like Cours---to Course manually
Note:
1.Icollection never have indexing if you need change it to Ilist
2.Virtual use for Lazy Loading
3.Initialised List Proeprty like
public Tag(){

Courses=new HashSet<Course>(); //why Hashset needs to find
}


step-6
enable-migartions //if not enabled
add-migration InitialModel
add-migartion initialModel -IgnoreChanges -force
update-database //reuired to run empty migration to keeptarck of migrations


Note--Always make small changes and small migrations

//Adding a new class in code first

step-1 add class :Category:CategoryId for primary key
step-2 add Dbset<category> in DbCOntext class
step-2 add-migartion AddCategoriesTable ;//Datacentric Name
step-3 update-database

Note:By Default Id is identity if its int you can override it using identity:false



//Modifying an existing class in code first
1.adding new property
2.modifying property
3.deleteing property

1.Adding category
step-1 
add-migration addCategoryColumnToCourses
update-database

2.Modifying
step-1 add-migration renameTitleToName

stepe-2  Go To Migration add below method
delete add  methods and below
renameColumn("dbo.course","oldcolumnname","newcolumnName")

or
add name column--migrate  and then sql for update
sql("update Courses set name=title")
dropcolumn(ttilt)



3.Delete 
step-1 Remove from model
step-2 do migartions


//Deleting an Existing Class
step-1
Delete the property IF it is there

step-2 execute migration command

step-3 Remove class and Remove Dbset reference 
step-4 add-migration migrationname
step-5 Take backup of Data by copying createtable method from down and then sql to transfer data
step-5 update-database

//Downgrading Database
update-database -TargetMigration:DeleteCategoryColumn

//to go back at the current version
update-database


//Seeding Database
step-1 configuration.cs>seed()
{
  _context.Customers.AddOrUpdate(p=>P.Name,new Author{Name="xyz"})
}

step-2 update-database
Every time you run update database then seed methods gets triggered



//Override code-first:Overview
one way is using Data Annotations like [Required][StringLength(50)]

another using Fluent-API

step-1 Go to PlutoContextClass

step-2 Look for OnModelCreating
protected override OnModelCreating(DbModelBuilder modelBuilder){

modelBuilder.Entity<Customer>()
.Property(t=>t.Description)
.IsRequired();

}

step-3 do migration stuffs


//Data Annotations more in details
[MaxLength(255)]
[Required]
[Table("")]
[Column("")]
[Key]
[DatabaseGenerated(DatabaseGeneratedOption.None|Identity|Computed)]
[key]
[Column(order=1)]
x
[Key]
[Column(order=2)]
y

to make composite Key

[Index(Isuniue=true)]

[Index("k1",1)]
x
[Index("k2",2)]
y

//Foregn Key in Data Annotations

public class Course{

[ForeignKey("AuthorPropNm")]
public int AuthorId{get;set;}

public Author AuthorPropNm{get;set;}

}


//Implement FluentApi


protected override OnModelCreating(DbModelBuilder modelBuilder){

modelBuilder.Entity<Customer>()
.Property(t=>t.Description)
.IsRequired();

modelBuilder.Entity<Customer>()
.ToTable("TbleNm")

modelBuilder.Entity<Customer>()
.ToTable("TbleNm","catalogue");//change schema

modelBuilder.Entity<Customer>()
.HasKey(t=>t.ISBN);//primarykey

modelBuilder.Entity<Customer>()
.HasKey(t=> new{t.oid,t.itemId})//composite key

modelBuilder.Entity<Customer>()
.Property(t=>t.Description)
.HasCoulmnName("desc")
.HasColumnType("Varchar")
.HasColumnOrder(2)
.HasDatabaseGeneratedOption(DatabaseGeneratedOption.None)
.HasMAxLength(255);


}


//RelationShips in FluentApi

//HasMany(),HasRequired(),HasOptional()----->Right to left
//WithMany(),WithRequired(),WithOptional()  <-----Left to Right

//one to many Autho------* Course
//An author can hav emany courses
protected override OnModelCreating(DbModelBuilder modelBuilder){


1:Many
modelBuilder.Entity<Author>()
.HasMany(c=>c.courses)
.WithRequired(c=>c.Author)
.HasForeginKey(c=>c.AuthorId)


Many:Many
//course*----*tags
modelBuilder.Entity<Course>()
.HasMany(c=>c.Tags)
.WithMany(c=>c.courses)
.Map(m=>m.ToTable("CourseTags"))

1-----0:1
Course----Caption
modelBuilder.Entity<Course>()
.HasOptional(c=>c.Caption)
.WithRequired(c=>c.Course)

1----1
Course---Cover
Principle----Dependent
modelBuilder.Entity<Course>()
.HasRequired(c=>c.Cover)
.WithRequiredPrinciple(c=>c.Course)

or
.WithRequiredPrinciple(c=>c.Cover)







}