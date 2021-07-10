```C#
using UnityEngine;
using System.Collections;  
  
[[RequireComponent](https://docs.unity3d.com/ScriptReference/RequireComponent.html)(typeof([AudioSource](https://docs.unity3d.com/ScriptReference/AudioSource.html)))]
public class ExampleClass : [MonoBehaviour](https://docs.unity3d.com/ScriptReference/MonoBehaviour.html)
{
    public [AudioClip](https://docs.unity3d.com/ScriptReference/AudioClip.html) impact;
    [AudioSource](https://docs.unity3d.com/ScriptReference/AudioSource.html) audioSource;  
  
    void Start()
    {
        audioSource = GetComponent<[AudioSource](https://docs.unity3d.com/ScriptReference/AudioSource.html)>();
    }  
  
    void OnCollisionEnter()
    {
        audioSource.PlayOneShot(impact, 0.7F);
    }
}```
Play [[Audio]] in [[Unity3d]]