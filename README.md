# xlr-jython-code-snippets
XLR code snippets using python and jython API

### Get previous task

```
def previousTask(task):
  index = 0
  for item in phase.tasks:
    if item.id == task.id:
      break
    index = index + 1
  return task.getPhase().tasks[index-1]
print task.title
print "previous task is: " + str(previousTask(task).title)
```

### Get Gated tasks before a task

```
def gatesBeforeTask(task):
  gatesList = []
  for item in phase.tasks:
    if str(item.getTaskType()) == "xlrelease.GateTask":
     gatesList.append(item)
    if item.id == task.id:
     break
  return gatesList

print "start gates " + str(phase.getStartGates())
for item in gatesBeforeTask(task):
  print item.title
```

### Find a task by title

```
def findTaskByTitle(title):
  list = []
  for item in phase.tasks:
    if item.title == title:
      list.append(item)
  return list
```
### Check/uncheck a gate condition via REST API

\# Sample call:  python gateupdate.py Release546450 UAT 'Gate 2' 'Gate 2 condition 1' 'true' 

```
import requests
import sys

releaseId = sys.argv[1]
phaseTitle = sys.argv[2]
gateTitle = sys.argv[3]
conditionTitle = sys.argv[4]
conditionChecked = sys.argv[5]

release = requests.get('http://admin:xlradmin@localhost:5516/releases/' + releaseId)
for phase in release.json()['phases']:
  if phase['title'] == phaseTitle:
    for task in phase['tasks']:
      if task['title'] == gateTitle:
        for condition in task['conditions']:
          if condition['title'] == conditionTitle:
            condition['checked'] = conditionChecked
            r = requests.put('http://admin:xlradmin@localhost:5516/gates/conditions/' + condition['id'], json=condition)
            print r.status_code	
```


### Fetch variables from another release

```
rvar = releaseApi.getVariables(releaseVariables['myrel'])
for r in rvar: 
    print r._delegate.key
    print r._delegate.value

```

### Update automated task server entry

```
t = taskApi.searchTasksByTitle("buildjar","Build",release.id)[0]
jenkinslist = configurationApi.searchByTypeAndTitle("jenkins.Server",releaseVariables['jenkinsserver'])
t.pythonScript.jenkinsServer = jenkinslist[0]
taskApi.updateTask(t)
```

### Fetch URL for selected server in a automated task

```
t = taskApi.searchTasksByTitle("buildjar","Build",release.id)
jenkinsUrl = t[0].pythonScript.jenkinsServer.url
releaseVariables['jarurl'] = "%s/job/buildjar/%s/artifact/abc.jar" % (jenkinsUrl, releaseVariables['buildjarbn'])
```

### Create table/fancy markdown in a task description

```
a = [1,2,3,4,5,6]
out=  "|Heading \n |---\n"
for item in a:
    out= "{0}|{1}\n".format(out,item)
task.description = out
taskApi.updateTask(task)
```
#### Image
![images/tableintaskdescription.png](images/tableintaskdescription.png)



## Xlrelease add dynamic tags

```
release.tags.add('abc')
releaseApi.updateRelease(release)
```



## Print classpath 

```
from java.lang import ClassLoader
cl = ClassLoader.getSystemClassLoader()
paths = map(lambda url: url.getFile(), cl.getURLs())
print paths
```

## Add a folder

```
from com.xebialabs.xlrelease.domain.folder import Folder
newFolder = Folder()
newFolder.title="MyFolder"
folderApi.addFolder("Applications",newFolder)
```

## Copy a template

```
templates = folderApi.getTemplates("Applications")
# Pick the first template 
template = templates[0]
# Give it a new title
template.title = "new template"
templateApi.createTemplate(template)
```

## Generate a release pipeline dynamically with parallel group and tasks from a provided dataset

```
import json
teamProjectName = "MyProject"
buildDefinitionName = "mybuildDef"
datamap = '''[
	{
		"name": "libs",
		"entries": [
			{
				"name": "a-library",
				"skip": "false"
			},
			{
				"name": "b-library",
				"skip": "false"
			},
			{
				"name": "c-library",
				"skip": "true"
			}
		],
		"execution": "sequence"
	},
	{
		"name": "messages",
		"entries": [
			{
				"name": "a-messages",
				"skip": "false"
			},
			{
				"name": "b-messages",
				"skip": "true"
			},
			{
				"name": "c-messages",
				"skip": "false"
			}
		],
		"execution": "parallel"
	},
	{
		"name": "services",
		"entries": [
			{
				"name": "a-service",
				"skip": "false"
			},
			{
				"name": "b-service",
				"skip": "false"
			}
		],
		"execution": "parallel"
	}
]
'''
dataobj = json.loads(datamap)

for item in dataobj:
    phase = phaseApi.newPhase(item['name'])
    phase = phaseApi.addPhase(release.id, phase)
    if str(item['execution']) == "parallel":
        pgrouptask = taskApi.newTask("xlrelease.ParallelGroup")
        pgrouptask.title = "Parallel Run"
        phase = taskApi.addTask(phase.id, pgrouptask)
    for entry in item['entries']:
        task = taskApi.newTask("vsts.QueueBuild")
        task.title = entry['name']

        task.pythonScript.setProperty("teamProjectName", teamProjectName)
        task.pythonScript.setProperty("buildDefinitionName", buildDefinitionName)
        
        if entry['skip'] == "true":
            task.precondition = "True == False"
        taskApi.addTask(phase.id, task)
        
 ```

## Add a Custom task (MyCustomTask.TestMapOutPut) in curent running Release and set a Key-Value (dicoPackageMap) in output property (MapOut)

```
curRelease = getCurrentRelease()
curPhase = getCurrentPhase().title
task = taskApi.newTask("MyCustomTask.TestMapOutPut")
task.title = "ScriptCreated"
task.variableMapping = {"pythonScript.MapOut" : "$""{dicoPackageMap}"}
createdTask=taskApi.addTask(getCurrentPhase().id, task)
taskApi.assignTask(createdTask.id,"Admin")

```


