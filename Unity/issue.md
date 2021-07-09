```C#
public void Bind(List<GameFlagData\> gameFlagDatas)  
{  
 foreach (var flag in \_allFlags)  
 { var data = gameFlagDatas.FirstOrDefault(t => t.Name \== flag.name);  
 if (data == null)  
 { data = new GameFlagData() {Name \= flag.name};  
 gameFlagDatas.Add((data));  
 }  
  
 flag.Bind(data);  
 }  
}

```

