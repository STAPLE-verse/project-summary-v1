---
title: Contributors
toc: false
---

<link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<link rel="stylesheet" href="https://cdn.datatables.net/1.13.6/css/jquery.dataTables.min.css">
<script src="https://cdn.datatables.net/1.13.6/js/jquery.dataTables.min.js"></script>
<link rel="stylesheet" href="https://cdn.datatables.net/buttons/2.4.1/css/buttons.dataTables.min.css">
<script src="https://cdn.datatables.net/buttons/2.4.1/js/dataTables.buttons.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.1.3/jszip.min.js"></script>
<script src="https://cdn.datatables.net/buttons/2.4.1/js/buttons.html5.min.js"></script>
<link rel="stylesheet" href="style.css">

<div class="hero">
 <h1>Contributor Information</h1>
</div>

```js redirect
if (localStorage.getItem("jsonData") == null) {
  window.location.href = '/';
}
```

```js data
//data
const jsonData = JSON.parse(localStorage.getItem("jsonData"))
```

```js import-packages
import * as Plot from "npm:@observablehq/plot";
import * as d3 from "npm:d3";
import * as Inputs from "npm:@observablehq/inputs";
import Plotly from "npm:plotly.js-dist";
```

```js functions
function createHtmlTable(data, containerId, tableId) {
  const container = document.getElementById(containerId);

  if (!container) {
    console.error(`Container with id "${containerId}" not found.`);
    return;
  }

  // Create the table
  const table = document.createElement("table");
  table.id = tableId; // Assign the provided tableId
  table.className = "custom-table";

  // Create table header
  const thead = document.createElement("thead");
  const headerRow = document.createElement("tr");

  // Use the keys of the first object in the data as headers
  const headers = Object.keys(data[0]);
  headers.forEach((key) => {
    const th = document.createElement("th");
    th.textContent = key.replace(/([A-Z])/g, " $1"); // Add spaces before uppercase letters
    headerRow.appendChild(th);
  });

  thead.appendChild(headerRow);
  table.appendChild(thead);

  // Create table body
  const tbody = document.createElement("tbody");

  data.forEach((item) => {
    const row = document.createElement("tr");
    headers.forEach((key) => {
      const td = document.createElement("td");
      td.textContent = item[key] || "N/A"; // Fallback to "N/A" if the value is null/undefined
      row.appendChild(td);
    });
    tbody.appendChild(row);
  });

  table.appendChild(tbody);

  // Clear any existing content in the container and add the table
  container.innerHTML = "";
  container.appendChild(table);
}

const getComputedThemeColors = () => {
  const root = getComputedStyle(document.documentElement);
  return {
    primary3: root.getPropertyValue("--primary-color-3").trim(),
    primary4: root.getPropertyValue("--primary-color-4").trim(),
    secondary: root.getPropertyValue("--secondary-color").trim(),
  };
};

const themeColors = getComputedThemeColors();
```

```js create-donut-chart-function
function createDonutChart(data, containerId, titleText) {
  // Prepare data for Plotly donut chart
  const chartData = [
    {
      values: data.values, // Array of counts (e.g., tasks completed vs. remaining)
      labels: data.labels, // Array of labels (e.g., ["Completed", "Remaining"])
      type: "pie",
      hole: 0.5, // Donut chart effect
      textinfo: "none", // Hide text in the center
      hoverinfo: "label+percent", // Show labels and percentages on hover
      marker: {
        colors: data.colors || d3.schemeCategory10, // Provide colors or use default scheme
      },
    },
  ];

  // Layout for the chart
  const chartLayout = {
    //title: {
    //  text: titleText, // Title for the chart
    //  font: { size: 16 },
    //},
    showlegend: false, // Show legend
    height: 100, // Adjust chart height
    width: 100, // Adjust chart width
    margin: { t: 10, b: 10, l: 10, r: 10 }, // Add padding for the chart
    plot_bgcolor: "rgba(0, 0, 0, 0)", // Transparent plot background
    paper_bgcolor: "rgba(0, 0, 0, 0)", // Transparent paper background
  };

  // Render the chart in the specified container
  Plotly.newPlot(containerId, chartData, chartLayout);
}
```

```js get-people-roles-tasks
// Extract project members where there is exactly one user in the `users` array
const filteredMembers = jsonData.projectMembers.filter(
  (member) => member.name === null
);

// Map the data into a dataframe-like array of objects
const projectMembersDataFrame = filteredMembers.map((member) => {
  const user = member.users[0];
  const roles = member.roles.map((role) => role.name).join(", ");
  return {
    projectMemberId: member.id,
    username: user.username || "Not Provided",
    firstName: user.firstName || "Not Provided",
    lastName: user.lastName || "Not Provided",
    roles: roles || "No Roles",
    createdAt: member.createdAt,
    updatedAt: member.updatedAt,
  };
});

// Extract project members where there is more than one user in the `users` array
const groupMembers = jsonData.projectMembers.filter(
  (member) => member.name !== null
);

// Map the data into a dataframe-like array of objects, each user gets their own row
const groupMembersDataFrame = groupMembers.flatMap((member) => {
  return member.users.map((user) => ({
    projectMemberId: member.id,
    groupName: member.name || "Unnamed Group",
    username: user.username || "Not Provided",
    firstName: user.firstName || "Not Provided",
    lastName: user.lastName || "Not Provided",
    roles: member.roles.map((role) => role.name).join(", ") || "No Roles",
    createdAt: member.createdAt,
    updatedAt: member.updatedAt,
  }));
});

// Extract tasks data and expand to include task logs
const tasksDataFrame = jsonData.tasks.flatMap((task) =>
  task.taskLogs
    //.filter((log) => log.completedById !== null) // Skip logs where completedById is null
    .map((log) => ({
      taskId: task.id,
      createdAt: task.createdAt,
      updatedAt: task.updatedAt,
      createdById: task.createdById,
      formVersionId: task.formVersionId || "Not Provided",
      deadline: task.deadline || "No Deadline",
      name: task.name || "Unnamed Task",
      description: task.description || "No Description",
      status: task.status || "Unknown Status",
      elementName: task.element?.name || "No Element Name",
      elementDescription: task.element?.description || "No Element Description",
      taskLogCreatedAt: log.createdAt,
      taskLogStatus: log.status || "Unknown Status",
      taskLogMetadata: log.metadata || "No Metadata",
      completedById: log.completedById || "Task Created",
      assignedToId: log.assignedToId,
      roles: task.roles.map((role) => ({
        id: role.id,
        name: role.name,
        description: role.description,
        taxonomy: role.taxonomy,
      })),
    }))
);

// Extract all roles from project members
const memberRoles = jsonData.projectMembers.flatMap((member) => member.roles);

// Extract all roles from tasks
const taskRoles = tasksDataFrame.flatMap((task) => task.roles);

// Combine all roles from both sources
const allRoles = [...memberRoles, ...taskRoles];

// Deduplicate roles based on `name`, `description`, and `taxonomy`
const uniqueRolesDataFrame = Array.from(
  new Map(
    allRoles.map((role) => [
      `${role.name}-${role.description}-${role.taxonomy}`, // Unique key for deduplication
      role
    ])
  ).values()
);

// Transform the unique roles into a dataframe-like structure
const rolesDataFrame = uniqueRolesDataFrame.map((role) => ({
  name: role.name || "No Name",
  description: role.description || "No Description",
  taxonomy: role.taxonomy || "No Taxonomy",
}));
```

```js contributor-task-summaries
// Deduplicate task logs to only include the latest log for each combination of `taskId` and `assignedToId`
const latestTaskLogs = Array.from(
  tasksDataFrame
    .reduce((map, log) => {
      const key = `${log.taskId}-${log.assignedToId}`;
      const existingLog = map.get(key);

      // Keep the log with the latest `createdAt`
      if (!existingLog || new Date(log.taskLogCreatedAt) > new Date(existingLog.taskLogCreatedAt)) {
        map.set(key, log);
      }

      return map;
    }, new Map())
    .values()
);
// Group task logs by user and count completed tasks and metadata forms
const individualsWithTaskData = projectMembersDataFrame.map((individual) => {
  // Filter task logs completed by the individual
  const completedTaskLogs = latestTaskLogs.filter(
    (log) => log.assignedToId === individual.projectMemberId && log.taskLogStatus === "COMPLETED"
  );

  const allTaskLogs = latestTaskLogs.filter(
    (log) => log.assignedToId === individual.projectMemberId
  );

  // Count completed tasks
  const numberOfTasksCompleted = completedTaskLogs.length;
  const numberOfTasks = allTaskLogs.length;

  // Count metadata forms
  const numberOfMetadataForms = completedTaskLogs.filter(
    (log) => log.taskLogMetadata !== "No Metadata"
  ).length;

  const allMetaDataForms = allTaskLogs.filter(
    (log) => log.taskLogMetadata !== "No Metadata"
  ).length;

  // Construct name, replacing "Not Provided Not Provided" with "No Name Provided"
  const fullName =
    `${individual.firstName} ${individual.lastName}`.trim() === "Not Provided Not Provided"
      ? "No Name Provided"
      : `${individual.firstName} ${individual.lastName}`;

  return {
    projectMemberId: individual.projectMemberId,
    name: `${fullName} (${individual.username})`,
    numberOfTasksCompleted,
    numberOfTasks,
    numberOfMetadataForms,
    allMetaDataForms,
    username: individual.username,

  };
});
```

```js team-task-summaries
// Group task logs by team ID (assignedToId) and calculate completed tasks and metadata forms
const teamsWithTaskData = groupMembers.reduce((teamsMap, team) => {
  // Initialize or update team entry
  const teamData = teamsMap.get(team.id) || {
    teamId: team.id,
    teamName: team.name || "Unnamed Team",
    memberNames: [],
    numberOfTasksCompleted: 0,
    numberOfMetadataForms: 0,
  };

  // Add team member names
  team.users.forEach((user) => {
    const memberName = `${user.firstName?.trim() || "No Name"} ${user.lastName?.trim() || "Provided"} (${user.username || "No Username"})`;
    if (!teamData.memberNames.includes(memberName)) {
      teamData.memberNames.push(memberName);
    }
  });

  // Filter task logs completed by this team
  const completedTaskLogs = latestTaskLogs.filter(
    (log) => log.assignedToId === team.id && log.taskLogStatus === "COMPLETED"
  );

  // Update team-level task counts
  teamData.numberOfTasksCompleted += completedTaskLogs.length;
  teamData.numberOfMetadataForms += completedTaskLogs.filter(
    (log) => log.taskLogMetadata !== "No Metadata"
  ).length;

  // Store updated team data
  teamsMap.set(team.id, teamData);

  return teamsMap;
}, new Map());

// Convert the teams map to an array for rendering
const teamsDataArray = Array.from(teamsWithTaskData.values());
```

```js member-names
// Get full names
const uniqueMemberNames = new Set(
  individualsWithTaskData.map((individual) => individual.name)
);

// Count the unique members
const numberOfUniqueMembers = uniqueMemberNames.size;

const uniqueTeamNames = new Set(
  teamsDataArray.map((team) => team.name)
);
const numberOfUniqueTeams = teamsWithTaskData.size;
```

```js overall-tasks-statistics
// Count the total number of rows
const totalTasks = latestTaskLogs.length;

// Count the number of tasks with a status of "COMPLETED"
const completedTasks = latestTaskLogs.filter(log => log.taskLogStatus === "COMPLETED").length;
const completedPercentage = ((completedTasks / totalTasks) * 100).toFixed(1); // Calculate percentage

const dataCompletedTasks = [
  {
    values: [completedTasks, totalTasks - completedTasks],
    labels: ["Completed", "Remaining"],
    type: "pie",
    hole: 0.80, // Creates the donut effect
    textinfo: "none", // Hide default labels
    hoverinfo: "label+percent",
    marker: {
      colors: [themeColors.primary3, themeColors.secondary],
    },
  },
];

// Layout for the donut chart
const layout = {

  annotations: [
    {
      font: {
        size: 20,
        color: themeColors.primary3,
        weight: "bold", // Make the font bold
        family: "Arial, sans-serif",
      },
      showarrow: false,
      text: `${completedPercentage}%`, // Show percentage in the center
      x: 0.5,
      y: 0.5,
    },
  ],
showlegend: false, // Hide legend for simplicity
  height: 150, // Adjust height for the card
  width: 150, // Adjust width for the card
  margin: { t: 10, b: 10, l: 10, r: 10 }, // Tighten the chart's margins
  plot_bgcolor: "rgba(0, 0, 0, 0)", // Transparent plot background
  paper_bgcolor: "rgba(0, 0, 0, 0)", // Transparent chart area background

};

// Render the chart
Plotly.newPlot("completed-tasks-chart", dataCompletedTasks, layout);
```

```js overall-form-statistics
// Count the total number of form tasks
const totalForms = latestTaskLogs.filter(log => log.taskLogMetadata !== "No Metadata").length;

// Count the number of tasks with a status of "COMPLETED"
const completedForms = latestTaskLogs.filter(log => log.taskLogMetadata !== "No Metadata" && log.taskLogStatus === "COMPLETED").length;
const completedFormPercentage = ((completedForms / totalForms) * 100).toFixed(1); // Calculate percentage

const dataCompletedForms = [
  {
    values: [completedForms, totalForms - completedForms],
    labels: ["Completed", "Remaining"],
    type: "pie",
    hole: 0.80, // Creates the donut effect
    textinfo: "none", // Hide default labels
    hoverinfo: "label+percent",
    marker: {
      colors: [themeColors.primary4, themeColors.secondary],
    },
  },
];
// Layout for the donut chart
const layout = {
  annotations: [
    {
      font: {
        size: 20,
        color: themeColors.primary4,
        weight: "bold", // Make the font bold
        family: "Arial, sans-serif",
      },
      showarrow: false,
      text: `${completedFormPercentage}%`, // Show percentage in the center
      x: 0.5,
      y: 0.5,
    },
  ],
showlegend: false, // Hide legend for simplicity
  height: 150, // Adjust height for the card
  width: 150, // Adjust width for the card
  margin: { t: 10, b: 10, l: 10, r: 10 }, // Tighten the chart's margins
  plot_bgcolor: "rgba(0, 0, 0, 0)", // Transparent plot background
  paper_bgcolor: "rgba(0, 0, 0, 0)", // Transparent chart area background

};

// Render the chart
Plotly.newPlot("completed-forms-chart", dataCompletedForms, layout);
```

```js overall-role-statistics
// Step 1: Count roles in latestTaskLogs
const roleCountsFromTasks = latestTaskLogs.reduce((acc, log) => {
  log.roles.forEach((role) => {
    acc[role.name] = (acc[role.name] || 0) + 1; // Increment count for this role
  });
  return acc;
}, {});

// Step 2: Count roles in projectMembersDataFrame
const roleCountsFromMembers = projectMembersDataFrame.reduce((acc, member) => {
  member.roles.split(", ").forEach((role) => {
    acc[role] = (acc[role] || 0) + 1; // Increment count for this role
  });
  return acc;
}, {});

// Step 3: Merge counts into uniqueRolesDataFrame
const updatedRolesDataFrame = rolesDataFrame.map((role) => {
  const countFromTasks = roleCountsFromTasks[role.name] || 0;
  const countFromMembers = roleCountsFromMembers[role.name] || 0;

  return {
    ...role,
    countInTasks: countFromTasks, // Add count from latestTaskLogs
    countInMembers: countFromMembers, // Add count from projectMembersDataFrame
    totalCount: countFromTasks + countFromMembers, // Total occurrences
  };
});

// Extract labels (role names) and values (total counts) from updatedRolesDataFrame
const roleLabels = updatedRolesDataFrame.map((role) => role.name);
const roleValues = updatedRolesDataFrame.map((role) => role.totalCount);

// Data for Plotly donut chart
const roleData = [
  {
    values: roleValues,
    labels: roleLabels,
    type: "pie",
    hole: 0.5, // Creates the donut chart effect
    textinfo: "none", // Hide default labels (percentages or values inside the chart)
    hoverinfo: "label+percent", // Show detailed hover information
    marker: {
      colors: d3.schemeCategory10, // Use a predefined color scheme for variety
    },
  },
];

// Layout for the donut chart
const roleLayout = {
  showlegend: false, // Show legend
  height: 150, // Adjust chart height
  width: 150, // Adjust chart width
  margin: { t: 10, b: 10, l: 10, r: 10 }, // Add margins
  plot_bgcolor: "rgba(0, 0, 0, 0)", // Transparent plot background
  paper_bgcolor: "rgba(0, 0, 0, 0)", // Transparent paper background
};

// Render the donut chart
Plotly.newPlot("roles-donut-chart", roleData, roleLayout);
```

```js create-role-table
function createRoleTable() {
  // Create a table container dynamically
  const container = document.getElementById("role-table-container");
  if (!container) {
    console.error("Table container for roles not found!");
    return;
  }

  // Create the table element
  const table = document.createElement("table");
  table.id = "role-table";
  table.className = "display";

  // Clear the container and append the table
  container.innerHTML = "";
  container.appendChild(table);

  // Initialize the DataTable
  $("#role-table").DataTable({
    data: updatedRolesDataFrame,
    columns: [
      { data: "name", title: "Role Name", visible: true }, // Renamed column
      { data: "description", title: "Description", visible: true },
      { data: "taxonomy", title: "Taxonomy", visible: true },
      { data: "countInTasks", title: "Task Count", visible: true },
      { data: "countInMembers", title: "Member Count", visible: true },
      { data: "totalCount", title: "Total Count", visible: false },
    ],
    paging: true,
    searching: true,
    ordering: true,
    responsive: true,
    scrollX: true,
    dom: "frtipB", // Enable Buttons (B) in the DOM
    buttons: [
      {
        extend: "csvHtml5",
        text: "Download CSV",
        title: "Role_Data",
        className: "btn btn-primary",
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $('#role-table thead th').eq(columnIdx).text().trim();
            }
          },
        },
      },
      {
        extend: "excelHtml5",
        text: "Download Excel",
        title: "Role_Data",
        className: "btn btn-success",
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $('#role-table thead th').eq(columnIdx).text().trim();
            }
          },
        },
      },
    ],
    language: {
      search: "Search All: ", // Customize the label for the search box
    },
    initComplete: function () {
      // Optional: Add custom search inputs for each column
      this.api()
        .columns()
        .every(function () {
          const column = this;
          const header = $(column.header());
          const input = $('<input type="text" placeholder="Search ' + header.text() + '" />')
            .appendTo($(header).empty())
            .on("keyup change clear", function () {
              if (column.search() !== this.value) {
                column.search(this.value).draw();
              }
            });
        });
    },
  });
}

// Call the function to initialize the table
createRoleTable();
```

```js get-roles-individuals
const rolesByIndividual = projectMembersDataFrame.map((individual) => {
  // Extract roles assigned to the individual
  const individualRoles = individual.roles.split(", ").map((role) => ({
    name: role.trim(),
    source: "Assigned",
  }));

  // Extract roles from tasks completed by the individual
  const taskRoles = latestTaskLogs
    .filter((log) => log.assignedToId === individual.projectMemberId)
    .flatMap((log) => log.roles.map((role) => ({ name: role.name, source: "Task" })));

  // Combine and count role occurrences
  const roleCounts = [...individualRoles, ...taskRoles].reduce((acc, role) => {
    acc[role.name] = (acc[role.name] || 0) + 1;
    return acc;
  }, {});

  // Construct the individual's full name
  const fullName = `${individual.firstName?.trim() || "Not Provided"} ${
    individual.lastName?.trim() || "Not Provided"
  }`.trim();

  const formattedName =
    fullName === "Not Provided Not Provided"
      ? `No Name Provided (${individual.username || "No Username"})`
      : `${fullName} (${individual.username || "No Username"})`;

  // Find the corresponding individual task data
  const individualTaskData = individualsWithTaskData.find(
    (data) => data.projectMemberId === individual.projectMemberId
  );

  // Include completion data in the result
  return {
    projectMemberId: individual.projectMemberId,
    name: formattedName,
    roles: Object.entries(roleCounts).map(([roleName, count]) => ({
      name: roleName,
      count,
    })),
    numberOfTasksCompleted: individualTaskData?.numberOfTasksCompleted || 0,
    numberOfTasks: individualTaskData?.numberOfTasks || 0,
    numberOfMetadataForms: individualTaskData?.numberOfMetadataForms || 0,
    allMetaDataForms: individualTaskData?.allMetaDataForms || 0,
  };
});
```

```js make-donuts-roles-individuals
rolesByIndividual.forEach((member) => {
  // Data for the donut chart
  const data = {
    values: member.roles.map((role) => role.count),
    labels: member.roles.map((role) => role.name),
    colors: d3.schemeCategory10,
  };

  // Calculate completion percentages for tasks and forms
  const tasksPercentComplete = (
    (member.numberOfTasksCompleted / member.numberOfTasks) * 100
  ).toFixed(1);
  const formsPercentComplete = (
    (member.numberOfMetadataForms / member.allMetaDataForms) * 100
  ).toFixed(1);

  // Create the card container
  const card = document.createElement("div");
  card.className = "small-card clickable";

  // Add the member's name as a title
  const title = document.createElement("h3");
  title.textContent = member.name;
  card.appendChild(title);

  // Create a container for the donut chart
  const containerId = `member-role-chart-${member.projectMemberId}`;
  const chartDiv = document.createElement("div");
  chartDiv.id = containerId;
  card.appendChild(chartDiv);

  // Add progress bars for tasks and forms
  const progressBars = `
    <div class="progress-container">
      <div class="progress-bar" style="width: ${tasksPercentComplete}%; background-color: ${themeColors.primary3};" title="Tasks: ${tasksPercentComplete}% Completed"></div>
    </div>
    <div class="progress-container">
      <div class="progress-bar" style="width: ${formsPercentComplete}%; background-color: ${themeColors.primary4};" title="Forms: ${formsPercentComplete}% Submitted"></div>
    </div>
  `;
  card.innerHTML += progressBars; // Append progress bars to the card

  // Append the card to the members section and render the chart
  const memberRoleChartContainer = document.getElementById("members-section");
  if (memberRoleChartContainer) {
    memberRoleChartContainer.appendChild(card);
    createDonutChart(data, containerId, `${member.name}`);
  } else {
    console.error("Main container for member charts not found!");
  }

  // Event listener for details
  card.addEventListener("click", () => {
    showDetails("member", member.projectMemberId, member.name, "member-details-section");
  });
});
```

```js get-roles-teams
const rolesByTeam = teamsDataArray.map((team) => {
  const roleCounts = {};

  // Filter task logs where the task is assigned to this team
  const teamTaskLogs = latestTaskLogs.filter((log) => log.assignedToId === team.teamId);

  // Count roles across all tasks assigned to this team
  teamTaskLogs.forEach((log) => {
    log.roles.forEach((role) => {
      roleCounts[role.name] = (roleCounts[role.name] || 0) + 1;
    });
  });

  // Calculate task and form completion details
  const numberOfTasksCompleted = teamTaskLogs.filter((log) => log.taskLogStatus === "COMPLETED").length;
  const totalTasks = teamTaskLogs.length;
  const numberOfMetadataForms = teamTaskLogs.filter((log) => log.taskLogMetadata !== "No Metadata" && log.taskLogStatus === "COMPLETED").length;
  const totalForms = teamTaskLogs.filter((log) => log.taskLogMetadata !== "No Metadata").length;

  // Calculate percentages
  const tasksPercentComplete = ((numberOfTasksCompleted / totalTasks) * 100).toFixed(1);
  const formsPercentComplete = ((numberOfMetadataForms / totalForms) * 100).toFixed(1);

  // Return the team data with all required columns
  return {
    teamId: team.teamId,
    teamName: team.teamName,
    roles: Object.entries(roleCounts).map(([roleName, count]) => ({
      name: roleName,
      count,
    })),
    numberOfTasksCompleted: numberOfTasksCompleted || 0,
    totalTasks: totalTasks || 0,
    numberOfMetadataForms: numberOfMetadataForms || 0,
    totalForms: totalForms || 0,
    tasksPercentComplete: tasksPercentComplete || 0,
    formsPercentComplete: formsPercentComplete || 0,
  };
});

console.log(rolesByTeam)
```

```js make-donuts-teams
rolesByTeam.forEach((team) => {
  const data = {
    values: team.roles.map((role) => role.count),
    labels: team.roles.map((role) => role.name),
    colors: d3.schemeCategory10,
  };

  // Create the card container
  const card = document.createElement("div");
  card.className = "small-card clickable"; // Make it clickable

  // Add the team's name as a title
  const title = document.createElement("h3");
  title.textContent = team.teamName;
  card.appendChild(title);

  // Create a container for the donut chart
  const containerId = `team-role-chart-${team.teamId}`;
  const chartDiv = document.createElement("div");
  chartDiv.id = containerId;
  card.appendChild(chartDiv);

  // Add progress bars for tasks and forms
  const progressBars = `
    <div class="progress-container">
      <div class="progress-bar"
           style="width: ${team.tasksPercentComplete}%; background-color: ${themeColors.primary3};"
           title="Tasks: ${team.tasksPercentComplete}% Completed"></div>
    </div>
    <div class="progress-container">
      <div class="progress-bar"
           style="width: ${team.formsPercentComplete}%; background-color: ${themeColors.primary4};"
           title="Forms: ${team.formsPercentComplete}% Submitted"></div>
    </div>
  `;
  card.innerHTML += progressBars; // Append progress bars to the card

  // Append the card to the teams section and render the chart
  const teamRoleChartContainer = document.getElementById("teams-section");
  if (teamRoleChartContainer) {
    teamRoleChartContainer.appendChild(card);
    createDonutChart(data, containerId, `${team.teamName}`);
  } else {
    console.error("Main container for team charts not found!");
  }

  // Event listener for details
  card.addEventListener("click", () => {
    showDetails("team", team.teamId, team.teamName, "team-details-section");
  });
});
```

```js show-details-clicked-card
function showDetails(type, id, name, detailsSectionId) {
  const detailsSection = document.getElementById(detailsSectionId);

  if (!detailsSection) {
    console.error(`Details section with ID "${detailsSectionId}" not found.`);
    return;
  }

  // Clear existing content and add the header first
  detailsSection.innerHTML = `
    <h2>${name} Details</h2>
    <p>Here’s a summary of tasks and contributions for ${name}. The table contains the
    entire set of task logs for each person with the roles assigned to those task. You
    can download the table information by using the buttons at the bottom.</p>
  `;

  // Filter relevant data
  const relevantData =
    type === "member"
      ? tasksWithNames.filter((task) => task.assignedTo.includes(name))
      : tasksWithNames.filter((task) => {
          const team = groupMembersDataFrame.find((team) => team.projectMemberId === id);
          return team && task.assignedTo.includes(team.groupName);
        });

  if (relevantData.length === 0) {
    detailsSection.innerHTML += "<p>No tasks found for this selection.</p>";
    return;
  }

  // Create a container for the table (after the header)
  const tableContainer = document.createElement("div");
  tableContainer.id = `details-table-container-${id}`;

  // Append the container to the section
  detailsSection.appendChild(tableContainer);

  // Create and render the table within the container
  const tableId = `details-table-${id}`;
  createHtmlTable(relevantData, tableContainer.id, tableId);

  // Apply DataTables functionality
  $(`#${tableId}`).DataTable({
    paging: true,
    searching: true,
    ordering: true,
    responsive: true,
    scrollX: true,
    dom: "frtipB", // Enable Buttons (B) in the DOM
    buttons: [
      {
        extend: "csvHtml5",
        text: "Download CSV", // Customize button text
        title: "Roles_Data", // Name of the downloaded file
        className: "btn btn-primary", // Optional: Add a CSS class
        exportOptions: {
          columns: ':visible', // Export visible columns only
                    format: {
            header: function (data, columnIdx) {
              return $(`#${tableId}`).DataTable().settings().init().columns[columnIdx].title || '';
            }
          },
        },
      },
      {
        extend: "excelHtml5",
        text: "Download Excel",
        title: "Roles_Data", // Name of the downloaded file
        className: "btn btn-success", // Optional: Add a CSS class
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $(`#${tableId}`).DataTable().settings().init().columns[columnIdx].title || '';
            }
          },
        },
      },
    ],
      columns: [
    { data: "taskId", title: "Task Id", visible: false },
    { data: "createdAt", title: "Created Date", visible: true },
    { data: "updatedAt", title: "Updated Date", visible: true },
    { data: "createdById", title: "Created By Id", visible: false },
    { data: "formVersionId", title: "Form Version Id", visible: false },
    { data: "deadline", title: "Deadline", visible: true },
    { data: "name", title: "Task Name", visible: true },
    { data: "description", title: "Task Description", visible: true },
    { data: "status", title: "Task Completed", visible: true },
    { data: "elementName", title: "Element Name", visible: true },
    { data: "elementDescription", title: "Element Description", visible: true },
    { data: "taskLogCreatedAt", title: "Task Log Date", visible: true },
    { data: "taskLogStatus", title: "Task Log Completed", visible: true },
    { data: "taskLogMetadata", title: "Form Data", visible: true },
    { data: "completedById", title: "Completed By Id", visible: false },
    { data: "assignedToId", title: "Assigned To Id", visible: false },
    { data: "roles", title: "Roles", visible: true },
    { data: "assignedTo", title: "Assigned To", visible: true },
    { data: "completedBy", title: "Completed By", visible: true },
  ],
    language: {
      search: "Search All: ", // Customize the label for the search box
    },
    initComplete: function () {
      // Optional: Add custom search inputs for each column
      this.api()
        .columns()
        .every(function () {
          const column = this;
          const header = $(column.header());
          const input = $('<input type="text" placeholder="Search ' + header.text() + '" />')
            .appendTo($(header).empty())
            .on("keyup change clear", function () {
              if (column.search() !== this.value) {
                column.search(this.value).draw();
              }
            });
        });
    },
  });
};
```

```js tasks-with-names-formatted
const tasksWithNames = tasksDataFrame.map((task) => {
  // Find the assigned team or individual name
  const assignedName = (() => {
    const assignedTeam = groupMembersDataFrame.find(
      (team) => team.projectMemberId === task.assignedToId
    );

    if (assignedTeam) {
      return assignedTeam.groupName || "Unnamed Team";
    }

    const assignedMember = projectMembersDataFrame.find(
      (member) => member.projectMemberId === task.assignedToId
    );

    if (assignedMember) {
      const fullName =
        `${assignedMember.firstName?.trim() || "Not Provided"} ${
          assignedMember.lastName?.trim() || "Not Provided"
        }`.trim();
      return fullName === "Not Provided Not Provided"
        ? `No Name Provided (${assignedMember.username || "No Username"})`
        : `${fullName} (${assignedMember.username || "No Username"})`;
    }
    return "Unassigned"; // Fallback if no match is found
  })();

  // Find the completed by team or individual name
  const completedByName = (() => {
    if (task.completedById === "Task Created") {
      return "Task Created"; // Special case handling
    }

    const completedByTeam = groupMembersDataFrame.find(
      (team) => team.projectMemberId === task.completedById
    );

    if (completedByTeam) {
      return completedByTeam.groupName || "Unnamed Team";
    }

    const completedByMember = projectMembersDataFrame.find(
      (member) => member.projectMemberId === task.completedById
    );

    if (completedByMember) {
      const fullName =
        `${completedByMember.firstName?.trim() || "Not Provided"} ${
          completedByMember.lastName?.trim() || "Not Provided"
        }`.trim();
      return fullName === "Not Provided Not Provided"
        ? `No Name Provided (${completedByMember.username || "No Username"})`
        : `${fullName} (${completedByMember.username || "No Username"})`;
    }
    return "Unknown"; // Fallback if no match is found
  })();

  // Combine role names into a single string
  const combinedRoles = task.roles.map((role) => role.name).join(", ");

  // Format taskLogMetadata as a JSON string if it contains data
  const formattedMetadata =
    task.taskLogMetadata && typeof task.taskLogMetadata === "object"
      ? JSON.stringify(task.taskLogMetadata, null, 2) // Pretty-printed JSON
      : task.taskLogMetadata;

  // Convert taskLogStatus to a user-friendly format
  const formattedStatus =
    task.taskLogStatus === "COMPLETED" ? "Completed" : "Not Completed";

  const formattedTaskStatus =
    task.taskStatus === "COMPLETED" ? "Completed" : "Not Completed";

  // Return a new object with updated values
  return {
    ...task, // Spread the existing task data
    assignedTo: assignedName, // Replace assignedToId with the formatted name
    completedBy: completedByName, // Handle "Task Created" case and unknowns
    roles: combinedRoles, // Combine roles into a single string
    taskLogMetadata: formattedMetadata, // Pretty-print JSON if applicable
    taskLogStatus: formattedStatus, // User-friendly status
    status: formattedTaskStatus,
  };
});
```

```js combined-roles
// Combine roles by individual and team
const combinedRolesData = [
  ...rolesByIndividual.map((individual) => ({
    name: individual.name,
    type: "Individual",
    roles: individual.roles.map((role) => `${role.name} (${role.count})`).join(", "),
    tasksCompleted: individual.numberOfTasksCompleted || 0,
    totalTasks: individual.numberOfTasks || 0,
    formsSubmitted: individual.numberOfMetadataForms || 0,
    totalForms: individual.allMetaDataForms || 0,
  })),
  ...rolesByTeam.map((team) => ({
    name: team.teamName,
    type: "Team",
    roles: team.roles.map((role) => `${role.name} (${role.count})`).join(", "),
    tasksCompleted: team.numberOfTasksCompleted || 0,
    totalTasks: team.totalTasks || 0, // Default to 0 if missing
    formsSubmitted: team.numberOfMetadataForms || 0,
    totalForms: team.totalForms || 0, // Default to 0 if missing
  })),
];
```

```js download-datatable-roles
function createCombinedRolesTable() {
  // Create a table container dynamically
  const container = document.getElementById("roles-table-container");
  if (!container) {
    console.error("Table container for roles not found!");
    return;
  }

  // Create the table element
  const table = document.createElement("table");
  table.id = "combined-roles-table";
  table.className = "display";

  // Clear the container and append the table
  container.innerHTML = "";
  container.appendChild(table);

  // Initialize the DataTable
  $(table).DataTable({
    data: combinedRolesData,
    columns: [
      { data: "name", title: "Name" },
      { data: "type", title: "Type" },
      { data: "roles", title: "Roles" },
      { data: "tasksCompleted", title: "Tasks Completed" },
      { data: "totalTasks", title: "Total Tasks" },
      { data: "formsSubmitted", title: "Forms Submitted" },
      { data: "totalForms", title: "Total Forms" },
    ],
    paging: true,
    searching: true,
    ordering: true,
    responsive: true,
    scrollX: true,
    dom: "frtipB", // Enable Buttons
    buttons: [
      {
        extend: "csvHtml5",
        text: "Download CSV",
        title: "Combined_Roles_Data",
        className: "btn btn-primary",
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $('#combined-roles-table thead th').eq(columnIdx).text().trim();
            }
          },
        },
      },
      {
        extend: "excelHtml5",
        text: "Download Excel",
        title: "Combined_Roles_Data",
        className: "btn btn-success",
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $('#combined-roles-table thead th').eq(columnIdx).text().trim();
            }
          },
        },
      },
    ],
  language: {
    search: "Search All: ", // Customize the label for the search box
  },
  initComplete: function () {
    // Optional: Add custom search inputs for each column
    this.api()
      .columns()
      .every(function () {
        const column = this;
        const header = $(column.header());
        const input = $('<input type="text" placeholder="Search ' + header.text() + '" />')
          .appendTo($(header).empty())
          .on("keyup change clear", function () {
            if (column.search() !== this.value) {
              column.search(this.value).draw();
            }
          });
      });
  },
});
}

// Call this function to initialize the table
createCombinedRolesTable();
```

```js combined-task-data
// Combine task data for both members and teams
const combinedTaskData = tasksWithNames.map((task) => ({
  taskId: task.taskId,
  createdAt: task.createdAt || "N/A",
  updatedAt: task.updatedAt || "N/A",
  createdById: task.createdById || "N/A",
  formVersionId: task.formVersionId || "N/A",
  deadline: task.deadline || "No Deadline",
  name: task.name || "Unnamed Task",
  description: task.description || "No Description",
  status: task.status || "Unknown Status",
  elementName: task.elementName || "No Element Name",
  elementDescription: task.elementDescription || "No Element Description",
  taskLogCreatedAt: task.taskLogCreatedAt || "N/A",
  taskLogStatus: task.taskLogStatus || "Not Completed",
  taskLogMetadata: task.taskLogMetadata || "No Metadata",
  completedById: task.completedById || "N/A",
  assignedToId: task.assignedToId || "N/A",
  roles: task.roles || "No Roles",
  assignedTo: task.assignedTo || "Unassigned",
  completedBy: task.completedBy || "Unknown",
}));
```

```js combined-data-download
function createCombinedTaskTable() {
  const container = document.getElementById("combined-task-table-container");
  if (!container) {
    console.error("Table container for tasks not found!");
    return;
  }

  // Create the table element
  const table = document.createElement("table");
  table.id = "combined-task-table";
  table.className = "display";

  // Clear the container and append the table
  container.innerHTML = "";
  container.appendChild(table);

  // Initialize the DataTable
  $(table).DataTable({
    data: combinedTaskData,
    columns: [
      { data: "taskId", title: "Task Id", visible: false },
      { data: "createdAt", title: "Created Date", visible: true },
      { data: "updatedAt", title: "Updated Date", visible: true },
      { data: "createdById", title: "Created By Id", visible: false },
      { data: "formVersionId", title: "Form Version Id", visible: false },
      { data: "deadline", title: "Deadline", visible: true },
      { data: "name", title: "Task Name", visible: true },
      { data: "description", title: "Task Description", visible: true },
      { data: "status", title: "Task Completed", visible: true },
      { data: "elementName", title: "Element Name", visible: true },
      { data: "elementDescription", title: "Element Description", visible: true },
      { data: "taskLogCreatedAt", title: "Task Log Date", visible: true },
      { data: "taskLogStatus", title: "Task Log Completed", visible: true },
      { data: "taskLogMetadata", title: "Form Data", visible: true },
      { data: "completedById", title: "Completed By Id", visible: false },
      { data: "assignedToId", title: "Assigned To Id", visible: false },
      { data: "roles", title: "Roles", visible: true },
      { data: "assignedTo", title: "Assigned To", visible: true },
      { data: "completedBy", title: "Completed By", visible: true },
    ],
    paging: true,
    searching: true,
    ordering: true,
    responsive: true,
    scrollX: true,
    dom: "frtipB",
    buttons: [
      {
        extend: "csvHtml5",
        text: "Download CSV",
        title: "Combined_Tasks_Data",
        className: "btn btn-primary",
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $(table).DataTable().settings().init().columns[columnIdx].title || '';
            }
          },
        },
      },
      {
        extend: "excelHtml5",
        text: "Download Excel",
        title: "Combined_Tasks_Data",
        className: "btn btn-success",
        exportOptions: {
          columns: ':visible', // Export visible columns only
          format: {
            header: function (data, columnIdx) {
              return $(table).DataTable().settings().init().columns[columnIdx].title || '';
            }
          },
        },
      },
    ],
    language: {
      search: "Search All: ", // Customize the search label
    },
    initComplete: function () {
      // Optional: Add custom search inputs for each column
      this.api()
        .columns()
        .every(function () {
          const column = this;
          const header = $(column.header());
          const input = $('<input type="text" placeholder="Search ' + header.text() + '" />')
            .appendTo($(header).empty())
            .on("keyup change clear", function () {
              if (column.search() !== this.value) {
                column.search(this.value).draw();
              }
            });
        });
    },
  });
}

// Call the function to initialize the table
createCombinedTaskTable();
```

<div class ="card">

  <div class="card-title">
    <h1>Overall Statistics</h1>
  </div>

  <p>This page displays statistics and information about each contributor including tasks, task logs, and assigned roles. Click on each box to learn more and view the data. </p>

  <div class="statistics-container">
    <a href="#members" class="card-link">
      <div class="stat-card">
        <h3>Total Members</h3>
        <p id="stat-number-1">${numberOfUniqueMembers}</p>
        <i class="fas fa-users" id="stat-number-1" aria-hidden="true"></i>
      </div>
    </a>

  <a href="#teams" class="card-link">
    <div class="stat-card">
      <h3>Total Teams</h3>
      <p id="stat-number-2">${numberOfUniqueTeams}</p>
      <i class="fas fa-user-friends" id="stat-number-2" aria-hidden="true"></i>
    </div>
  </a>

  <a href="#roles" class="card-link">
    <div class="stat-card">
      <h3>Roles</h3>
      <p id="roles-donut-chart"></p>
    </div>
  </a>

  <a href="elements_tasks" class="card-link">
    <div class="stat-card">
      <h3>Tasks Completed</h3>
      <p id="completed-tasks-chart"></p>
    </div>
  </a>

  <a href="forms" class="card-link">
    <div class="stat-card">
      <h3>Forms Submitted</h3>
      <p id="completed-forms-chart"></p>
    </div>
  </a>

  </div>
</div>

<div class="custom-collapse">
  <input type="checkbox" class="toggle-checkbox" id="collapse-toggle-members">
  <label for="collapse-toggle-members" class="collapse-title">
    <div class="card-title" id="members"><h1>Members</h1></div>
    <i class="expand-icon">+</i>
  </label>
  <div class="collapse-content">
  <p>This section contains the role information for each member. Click on an individual card to learn more about their contribution to the project.</p>
    <div id="members-section"></div> <!-- Placeholder for dynamic content -->
    <div id="member-details-section" class="details-container">
      <p>Select a card to view details about the contributor or team.</p>
    </div>
  </div>
</div>

<div class="custom-collapse">
  <input type="checkbox" class="toggle-checkbox" id="collapse-toggle-teams">
  <label for="collapse-toggle-teams" class="collapse-title">
    <div class="card-title" id="teams"><h1>Teams</h1></div>
    <i class="expand-icon">+</i>
  </label>
  <div class="collapse-content">
    <p>This section contains the role information for each team. Click on an individual card to learn more about their contribution to the project.</p>
    <div id="teams-section"></div> <!-- Placeholder for dynamic content -->
    <div id="team-details-section" class="details-container">
      <p>Select a card to view details about the contributor or team.</p>
    </div>
  </div>
</div>

<div class="custom-collapse">
  <input type="checkbox" class="toggle-checkbox" id="collapse-toggle-roles">
  <label for="collapse-toggle-roles" class="collapse-title">
    <div class="card-title" id="roles"><h1>Roles</h1></div>
    <i class="expand-icon">+</i>
  </label>
  <div class="collapse-content">
    <p>This section contains the descriptive information of all roles, and the overall roles summarized across tasks and members.</p>
    <div id="role-table-container"></div>
  </div>
</div>

<div class="custom-collapse">
  <input type="checkbox" class="toggle-checkbox" id="collapse-toggle-roles-combined">
  <label for="collapse-toggle-roles-combined" class="collapse-title">
    <div class="card-title" id="roles-combined"><h1>Combined Roles Data</h1></div>
    <i class="expand-icon">+</i>
  </label>
  <div class="collapse-content">
    <p>This table includes combined roles data for both individuals and teams, along with task and form completion statistics. You can download it using the buttons below.</p>
    <div id="roles-table-container"></div> <!-- Placeholder for the table -->
  </div>
</div>

<div class="custom-collapse">
  <input type="checkbox" class="toggle-checkbox" id="collapse-toggle-tasks-combined">
  <label for="collapse-toggle-tasks-combined" class="collapse-title">
    <div class="card-title" id="tasks-combined"><h1>Combined Task Data</h1></div>
    <i class="expand-icon">+</i>
  </label>
  <div class="collapse-content">
    <p>This table includes task data for both individuals and teams, along with their assigned roles, statuses, and form data. You can download the information using the buttons below.</p>
    <div id="combined-task-table-container"></div> <!-- Placeholder for the table -->
  </div>
</div>
