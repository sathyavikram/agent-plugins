# agent-skills
agent-skills

This is project containing all agent skills I use.
Each agent skill should be in its own folder

Lets create first skill called free-cad-project-setup. This skill is used to setup freecad project and how python code for freecad should be written

- CAD files are generated using freecad python code
- Read readme.md file to requirements
- Always create multiple 3D CAD parts with file names starting part_number_name (e.g part_01_bin)
- all paramaters should be in config file
- All paramaters should scale with scale paramater
- every part file should have main function to run seperatelyt
- Every part code should export STEP and STL file to exports folder
- before creating step and stl files, delete existing file
- create 3 folders called - 3d-print, exports, media
- create assembly.py file. This file should import all part files and create final assembly including each part
- export assembly step and stl files to export folder
- assembly should delete all files in exports and run each part file
- assembly should use generated step files to create final assembly
- look at below projects to find common patterns and include as part of this skill

/Users/intelligentmachine/Documents/workspace/3d-models/catch-basin-drain-kit
/Users/intelligentmachine/Documents/workspace/3d-models/cord-storage-reel-v3
/Users/intelligentmachine/Documents/workspace/3d-models/carry-bag-bin

rm -rf exports && /Applications/FreeCAD.app/Contents/Resources/bin/freecadcmd assembly.py &&  /Applications/FreeCAD.app/Contents/MacOS/FreeCAD exports/assembly.step

/Applications/FreeCAD.app/Contents/Resources/bin/freecadcmd export_all.py
/Applications/FreeCAD.app/Contents/MacOS/FreeCAD exports/assembly.step    