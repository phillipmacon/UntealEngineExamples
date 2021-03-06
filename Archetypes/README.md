# WORK IN PROGRESS: Archetype Example

Investigation into some of ways UObject instances are used in terms of Class Default Objects and Archetypes.

What is a Class Default Object (CDO)?
What is an archetype?
What is a template (not C++ template, but in the context  of UObjects)?

In Engine/Source/Runtime/CoreUObject/Public/UObject/Object.h, EObjectFlags has `RF_ClassDefaultObject` and `RF_ArchetypeObject`:

```
enum EObjectFlags
{
...
	RF_ClassDefaultObject		=0x00000010,	///< This object is its class's default object
	RF_ArchetypeObject			=0x00000020,	///< This object is a template for another object - treat like a class default object
```

	/**
	* Determines whether this object is a template object
	*
	* @return	true if this object is a template object (owned by a UClass)
	*/
	bool IsTemplate( EObjectFlags TemplateTypes = RF_ArchetypeObject|RF_ClassDefaultObject ) const;

From [Objects](https://docs.unrealengine.com/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Objects/index.html):


> The UCLASS macro gives the UObject a reference to a UCLASS that describes its
> Unreal-based type. Each UCLASS maintains one Object called the 'Class Default
> Object', or CDO for short. The CDO is essentially a default 'template'
> Object, generated by the class constructor and unmodified thereafter. Both
> the UCLASS and the CDO can be retrieved for a given Object instance, though
> they should generally be considered read-only. The UCLASS for an Object
> instance can be accessed at any time using the GetClass() function.



### WIP:

## Question: What is a Class Default Object, Create Default Subobject, Archetype, and Template?


enum EObjectFlags
{
...
	RF_ClassDefaultObject		=0x00000010,	///< This object is its class's default object
	RF_ArchetypeObject			=0x00000020,	///< This object is a template for another object - treat like a class default object



gt
bool IsTemplate(UObject* Obj)
{
	return Obj &&
		(Obj->HasAnyFlags(RF_ClassDefaultObject | RF_ArchetypeObject) ||
		(Obj->HasAnyFlags(RF_DefaultSubObject) && Obj->GetOuter()->HasAnyFlags(RF_ClassDefaultObject | RF_ArchetypeObject)));
}

UObjectArchetype.cpp, Object.h


### Object.h
static UObject* GetArchetypeFromRequiredInfo(const UClass* Class, const UObject* Outer, FName Name, EObjectFlags ObjectFlags);
UObject* GetArchetype() const;
void GetArchetypeInstances( TArray<UObject*>& Instances );
void InstanceSubobjectTemplates( struct FObjectInstancingGraph* InstanceGraph = NULL );

/**
 * Calls PreEditChange on all instances based on an archetype in AffectedObjects. Recurses on any instances.
 *
 * @param	AffectedObjects		the array of objects which have this object in their ObjectArchetype chain and will be affected by the change.
 *								Objects which have this object as their direct ObjectArchetype are removed from the list once they're processed.
 */
void PropagatePreEditChange( TArray<UObject*>& AffectedObjects, FEditPropertyChain& PropertyAboutToChange );

/**
 * Calls PostEditChange on all instances based on an archetype in AffectedObjects. Recurses on any instances.
 *
 * @param	AffectedObjects		the array of objects which have this object in their ObjectArchetype chain and will be affected by the change.
 *								Objects which have this object as their direct ObjectArchetype are removed from the list once they're processed.
 */
void PropagatePostEditChange( TArray<UObject*>& AffectedObjects, FPropertyChangedChainEvent& PropertyChangedEvent );

/**
 * Serializes the script property data located at Data.  When saving, only saves those properties which differ from the corresponding
 * value in the specified 'DiffObject' (usually the object's archetype).
 *
 * @param	Ar				the archive to use for serialization
 */
void UObject::SerializeScriptProperties( FArchive& Ar ) const;

/**
 * Wrapper function for InitProperties() which handles safely tearing down this object before re-initializing it
 * from the specified source object.
 *
 * @param	SourceObject	the object to use for initializing property values in this object.  If not specified, uses this object's archetype.
 * @param	InstanceGraph	contains the mappings of instanced objects and components to their templates
 */
void ReinitializeProperties( UObject* SourceObject=NULL, struct FObjectInstancingGraph* InstanceGraph=NULL );

/**
 * Determine if this object has SomeObject in its archetype chain.
 */
inline bool IsBasedOnArchetype( const UObject* const SomeObject ) const;

### UObjectGlobals.h


struct FStaticConstructObjectParameters
{
	/** The class of the object to create */
	const UClass* Class;

	/** The object to create this object within (the Outer property for the new object will be set to the value specified here). */
	UObject* Outer;

	/** The name to give the new object.If no value(NAME_None) is specified, the object will be given a unique name in the form of ClassName_#. */
	FName Name;

	/** The ObjectFlags to assign to the new object. some flags can affect the behavior of constructing the object. */
	EObjectFlags SetFlags = RF_NoFlags;

	/** The InternalObjectFlags to assign to the new object. some flags can affect the behavior of constructing the object. */
	EInternalObjectFlags InternalSetFlags = EInternalObjectFlags::None;

	/** If true, copy transient from the class defaults instead of the pass in archetype ptr(often these are the same) */
	bool bCopyTransientsFromClassDefaults = false;

	/** If true, Template is guaranteed to be an archetype */
	bool bAssumeTemplateIsArchetype = false;

	/**
	 * If specified, the property values from this object will be copied to the new object, and the new object's ObjectArchetype value will be set to this object.
	 * If nullptr, the class default object is used instead.
	 */
	UObject* Template = nullptr;

	/** Contains the mappings of instanced objects and components to their templates */
	FObjectInstancingGraph* InstanceGraph = nullptr;

	/** Assign an external Package to the created object if non-null */
	UPackage* ExternalPackage = nullptr;

	COREUOBJECT_API FStaticConstructObjectParameters(const UClass* InClass);
};


