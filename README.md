# Power BI projects

This respository show examples of how to work with Power BI Projects (.pbip) files, to implement vestion control when teams are working with Power BI. Such a great way to colaborate in Data Visualization projects with this awesome tool.

In June 2023, Microsoft included Developer Mode1 in the Power BI environment as a solution to the high demand for business BI features that enable team collaboration and automation in development, testing, and production environments. This content was exclusive to Power BI licenses, but is now part of Power BI Desktop.

This is a feature characteristics available to activate through the option menu in Power BI desktop, once activate, you can save files under this option. Before to start, is important to considerate the next files type extensions you'll find in PBI universe.

- .tmdl: Tabular Model Definition Language
- .pbix: Power BI Report (Power BI Desktop Files)
- .pbip: Power BI Project
- .pbir: Power BI enhanced report format 

## Creating and saving Power BI projects

Creat a Power BI project is so easy, you can open an white Power BI desktop file, even without any data, you go to Save As, and then when you're searching where save it, it saving options you need to select the option _Power BI Project Files (*.pbip) in Type. Once accepted you'll see a folder structured by the next way:

```
|-- <project name>.SemanticModel: A collection of files and folders representing a Power BI semantic model.
|-- <project name>.Report: A collection of files and folders representing a Power BI report.
|-- .gitignore: Specifies intentionally untracked files that Git should ignore for Power BI project files, such as cache.abf and localSettings.json.
|-- <project name>.pbip: the desktop file ready to edit and work for your project.
```

Once all this elements are configured, you're ready to work and combine your skills in Power BI with git.
