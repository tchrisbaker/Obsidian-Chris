[[Jason Archetecture Course]] 
C#y [[event]] example
```C#
//Inspectable.cs
public static event Action <bool> InspectablesInRangeChanged;


// InspectionPanel.cs
void OnEnable()  
{ 
 
 Inspectable.InspectablesInRangeChanged += UpdateHintTextState;  
}  
  
void OnDisable() => Inspectable.InspectablesInRangeChanged -= UpdateHintTextState;