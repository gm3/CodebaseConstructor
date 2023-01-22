# CodebaseConstructor

This script is a Unity C# script that appears to be responsible for constructing code documentation. It uses regular expressions to search through all `C#` files in the `Application.dataPath` directory and extract information about classes, methods, and fields. The extracted information is then stored in a nested dictionary structure, where the top-level key is the class name, the second-level key is the method or field name, and the value is a list of strings containing information about the method or field. The script also includes a method called `GenerateMarkdown()` that appears to be responsible for generating markdown documentation from the extracted information, but the implementation is not shown. -GPT 

Certainly, here is a list of the methods, variables, and fields in the provided script and a general description of what they do:

Methods:

* `Start()`: This method is called when the script is first initialized. It gets a reference to the button and adds a listener for clicks, gets a list of all .cs files in the Application.dataPath directory, and creates a new directory in the `Application.streamingAssetsPath` directory. It then calls the TaskOnClick() method.
* `TaskOnClick()`: This method appears to be responsible for starting the process of generating code documentation. It calls the `SearchAllScripts()` method, which uses regular expressions to extract information about classes, methods, and fields from all C# files in the `Application.dataPath` directory.
* `SearchAllScripts()`: This method uses regular expressions to search through all C# files in the `Application.dataPath` directory and extract information about classes, methods, and fields. The extracted information is then stored in a nested dictionary structure, where the top-level key is the class name, the second-level key is the method or field name, and the value is a list of strings containing information about the method or field.
* `GenerateMarkdown()`: This method appears to be responsible for generating markdown documentation from the extracted information, but the implementation is not shown

Variables:

* `files`: This is an array of strings that contains a list of all .cs files in the Application.dataPath directory.
* `codebase`: This is a dictionary that stores the extracted information about classes, methods, and fields. The top-level key is the class name, the second-level key is the method or field name, and the value is a list of strings containing information about the method or field.
* `headingLevel`: This variable appears to control the level of headings used in the generated markdown documentation.


```csharp=
using System;
using System.IO;
using System.Text;
using System.Text.RegularExpressions;
using UnityEngine;
//using UnityEditor;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using UnityEngine.UI;
//using UnityEngine.JsonUtility;

public class CodeConstrutorGPT : MonoBehaviour
{
    //private Button button;
    private string[] files;
    private Dictionary<string, Dictionary<string, List<string>>> codebase;
    private int headingLevel = 2;

    private void Start()
    {
        // Get a reference to the button and add a listener for clicks
        //button = GetComponent<Button>();
        //button.onClick.AddListener(TaskOnClick);

        // Get a list of all .cs files in the Application.dataPath directory
        files = Directory.GetFiles(Application.dataPath, "*.cs", SearchOption.AllDirectories);

        // Create a new directory in the Application.streamingAssetsPath directory
        Directory.CreateDirectory(Application.streamingAssetsPath + "/Code_Output/");

        TaskOnClick();
    }

    private void SearchAllScripts()
    {
    codebase = new Dictionary<string, Dictionary<string, List<string>>>();
    var classNameRegex = new Regex(@"^\s*(public|private|protected)\s+class\s+(\w+)");
    var methodRegex = new Regex(@"^\s*(public|private|protected)\s+(.*)\s+(\w+)\s*\(");
    var fieldRegex = new Regex(@"^\s*(public|private|protected)\s+([^\s]+)\s+(\w+);");

    Parallel.ForEach(files, file =>
    {
        var lines = File.ReadAllLines(file);
        for (int i = 0; i < lines.Length; i++)
        {
            var match = classNameRegex.Match(lines[i]);
            if (match.Success)
            {
                var className = match.Groups[2].Value;
                lock (codebase)
                {
                    if (!codebase.ContainsKey(className))
                    {
                        codebase.Add(className, new Dictionary<string, List<string>>());
                    }
                }
                for (int j = i + 1; j < lines.Length; j++)
                {   
                    var matchMethod = methodRegex.Match(lines[j]);
                    if (matchMethod.Success)
                    {
                        var methodName = matchMethod.Groups[3].Value;
                        var methodType = matchMethod.Groups[1].Value;
                        var methodReturnType = matchMethod.Groups[2].Value;
                        lock (codebase[className])
                        {
                            if (!codebase[className].ContainsKey(methodName))
                            {
                                codebase[className].Add(methodName, new List<string>());
                            }
                            codebase[className][methodName].Add(methodType + " " + methodReturnType);
                        }

                        // new RegEx for private and public fields
                        //var fieldRegex = new Regex(@"^\s*(public|private)\s+(.*)\s+(.+);");
                        var matchField = fieldRegex.Match(lines[j]);
                        if (matchField.Success)
                        {
                            var fieldName = matchField.Groups[3].Value;
                            var fieldType = matchField.Groups[1].Value;
                            var fieldReturnType = matchField.Groups[2].Value;
                            lock (codebase[className])
                            {
                                if (!codebase[className].ContainsKey(fieldName))
                                {
                                    codebase[className].Add(fieldName, new List<string>());
                                }
                                codebase[className][fieldName].Add(fieldType + " " + fieldReturnType);
                            }
                        }
                    }
                
                }
            }
        }
    });
    }
        
    

    
    private string GenerateMarkdown()
{
    var markdown = new StringBuilder();
    foreach (var className in codebase.Keys)
    {
        markdown.Append(new string('#', headingLevel) + " " + className + "\n");
        foreach (var fieldName in codebase[className].Keys)
        {
            var fieldType = codebase[className][fieldName][0];
            if (fieldType == "public" || fieldType == "private")
            {
                markdown.Append("- " + fieldType + " " + fieldName + ";\n");
            }
            else
            {
                markdown.Append("- " + string.Join(",", codebase[className][fieldName]) + " " + fieldName + "\n");
            }
        }
    }
    return markdown.ToString();
}

    private void CreateTextFile(string fileName, string content)
    {
        try
        {
            File.WriteAllText(Application.streamingAssetsPath + "/Code_Output/" + fileName, content);
        }
        catch (Exception e)
        {
            Debug.LogError("Error writing file: " + e.Message);
        }
    }

    private void LogCodebase()
    {
        foreach (var className in codebase.Keys)
        {
            Debug.Log("Class: " + className);
            foreach (var methodName in codebase[className].Keys)
            {
                Debug.Log("Method: " + methodName);
                foreach (var method in codebase[className][methodName])
                {
                    Debug.Log("Signature: " + method);
                }
            }
        }
    }

    private void TaskOnClick()
    {
        SearchAllScripts();
        LogCodebase();
        Debug.Log(codebase);
        string json = JsonUtility.ToJson(codebase);
        //Debug.Log(json);
        File.WriteAllText(Application.streamingAssetsPath + "/Code_Output/codebase.json", json);
        var markdown = GenerateMarkdown();
        CreateTextFile("codebase.md", markdown);
    }
}
```
