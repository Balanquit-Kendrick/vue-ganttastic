<template>
  <table class="data">
    <thead>
      <tr>
        <th class="title">Title</th>
        <th class="manager">Manager</th>
        <th class="label">Label</th>
      </tr>
    </thead>
    <tbody>
      <!-- Loop through flattened data -->
      <tr v-for="(item, index) in flattenedData" :key="index">
        <td :style="{ paddingLeft: `${item.level * 20}px` }" class="title">
          {{ item.title }}
        </td>
        <td class="manager">{{ item.type === 'task' ? item.managers.join(', ') : '' }}</td>
        <td class="label">{{ item.type === 'task' ? item.labels.join(', ') : '' }}</td>
      </tr>
    </tbody>
  </table>
  <!-- <div class="task-title">
    <div class="title">Title</div>
    <div class="manager">担当者</div>
    <div class="label">ラベル</div>
  </div>
  <div class="task-component">
    <div class="title">Folder</div>
    <div class="manager">担当者</div>
    <div class="label">ラベル</div>
  </div> -->
  <g-gantt-chart
    class="gantt-chart"
    :chart-start="chartStart"
    :chart-end="chartEnd"
    precision="day"
    :row-height="28"
    grid
    current-time
    width="100%"
    bar-start="beginDate"
    bar-end="endDate"
    :date-format="format"
    @click-bar="onClickBar($event.bar, $event.e, $event.datetime)"
    @mousedown-bar="onMousedownBar($event.bar, $event.e, $event.datetime)"
    @dblclick-bar="onMouseupBar($event.bar, $event.e, $event.datetime)"
    @mouseenter-bar="onMouseenterBar($event.bar, $event.e)"
    @mouseleave-bar="onMouseleaveBar($event.bar, $event.e)"
    @dragstart-bar="onDragstartBar($event.bar, $event.e)"
    @drag-bar="onDragBar($event.bar, $event.e)"
    @dragend-bar="onDragendBar($event.bar, $event.e, $event.movedBars)"
    @contextmenu-bar="onContextmenuBar($event.bar, $event.e, $event.datetime)"
  > 
    <g-gantt-row label="My row to test" :bars="bars1" highlight-on-hover />
    <g-gantt-row label="My another new row to test" highlight-on-hover :bars="bars2" />
    <g-gantt-row label="My another new row to test" highlight-on-hover :bars="bars2" />
    <g-gantt-row label="just another row to test gantt" highlight-on-hover :bars="bars3" />
    <g-gantt-row
      label="errors teach us, and debugging makes us stronger!"
      highlight-on-hover
      :bars="bars4"
    />
  </g-gantt-chart>
</template>

<script setup lang="ts">
import { ref, watch, computed } from "vue"
import type { GanttBarObject } from "./types"
import dayjs from "dayjs"

function formatDate(date: string): string {
  return dayjs(date, format.value).startOf("day").format(format.value);
}

interface Label {
  label_id: string;
  name: string;
}

interface Manager {
  user_id: string;
  name: string;
}

interface Task {
  task_id: string;
  name: string;
  labels?: Label[];
  managers?: Manager[];
}

interface Board {
  board_id: string;
  name: string;
  tasks?: Task[];
}

interface Room {
  room_id: string;
  name: string;
  boards?: Board[];
}

interface Folder {
  name: string;
  rooms: Room[];
}

interface FlattenedItem {
  title: string;
  type: 'folder' | 'room' | 'board' | 'task';
  level: number; // Indentation level
  managers?: string[];
  labels?: string[];
}

const flattenedData = computed<FlattenedItem[]>(() => {
      const data: FlattenedItem[] = []; // Explicitly type the 'data' array
      folder.value.forEach((folder) => {
        // Add folder row
        data.push({
          title: folder.name,
          type: 'folder',
          level: 0,
        });

        folder.rooms.forEach((room) => {
          // Add room row
          data.push({
            title: `${room.name}`,
            type: 'room',
            level: 1,
          });

          if (room.boards) {
            room.boards.forEach((board) => {
              // Add board row
              data.push({
                title: `${board.name}`,
                type: 'board',
                level: 2,
              });

              if (board.tasks) {
                board.tasks.forEach((task) => {
                  // Add task row
                  data.push({
                    title: `${task.name}`,
                    type: 'task',
                    level: 3,
                    managers: task.managers
                      ? task.managers.map((manager) => manager.name)
                      : [], // Handle missing managers
                    labels: task.labels
                      ? task.labels.map((label) => label.name)
                      : [], // Handle missing labels
                  });
                });
              }
            });
          }
        });
      });
      return data;
    });

const format = ref("DD.MM.YYYY HH:mm")
const chartStart = ref(dayjs().startOf("month").format(format.value))
const chartEnd = ref(dayjs().endOf("month").format(format.value))
// const chartEnd = ref(
//   dayjs(chartStart.value, format.value).add(3, "days").hour(12).format(format.value)
// )
const folder = ref<Folder[]>([
      {
        name: 'Test Folder',
        rooms: [
          {
            room_id: '111',
            name: 'Test Room 1',
            boards: [
              {
                board_id: '121',
                name: 'Test Board 1',
                tasks: [
                  {
                    task_id: '211',
                    name: 'Test Task 1',
                    labels: [
                      { label_id: '111', name: '1' },
                      { label_id: '121', name: '2' },
                    ],
                    managers: [
                      { user_id: '1234', name: '1' },
                      { user_id: '1234', name: '2' },
                    ],
                  },
                  {
                    task_id: '221',
                    name: 'Test Task 2',
                    labels: [
                      { label_id: '111', name: '1' },
                      { label_id: '121', name: '2' },
                    ],
                    managers: [
                      { user_id: '1234', name: '1' },
                      { user_id: '1234', name: '2' },
                    ],
                  },
                ],
              },
              {
                board_id: '122',
                name: 'Test Board 2',
                tasks: [
                  { task_id: '212', name: 'Test Task 1' },
                  { task_id: '222', name: 'Test Task 2' },
                ],
              },
            ],
          },
          {
            room_id: '222',
            name: 'Test Room 2',
          },
        ],
      },
    ]);

const bars1 = ref<GanttBarObject[]>([
  {
    beginDate: dayjs().startOf("day").format(format.value),
    endDate: dayjs().add(2, "day").endOf("day").format(format.value),
    ganttBarConfig: {
      id: "8621987329",
      label: "I'm in a bundle",
      bundle: "bundle2"
    }
  }
])

const bars2 = ref([
  {
    beginDate: dayjs().hour(13).startOf("hour").format(format.value),
    endDate: dayjs().hour(19).startOf("hour").format(format.value),
    ganttBarConfig: {
      id: "1592311887",
      label: "I'm in a bundle",
      bundle: "bundle2",
      style: {
        background: "magenta"
      }
    }
  },
  {
    beginDate: "15.01.2025 00:00",
    endDate: dayjs().add(2, "day").hour(19).startOf("hour").format(format.value),
    ganttBarConfig: {
      id: "7716981641",
      label: "Lorem ipsum dolor",
      hasHandles: true,
      style: {
        background: "#b74b52"
      }
    }
  },
  {
    beginDate: dayjs().add(1, "day").hour(4).startOf("hour").format(format.value),
    endDate: dayjs().add(1, "day").hour(16).startOf("hour").format(format.value),
    ganttBarConfig: {
      id: "9716981641",
      label: "Oh hey",
      style: {
        background: "#69e064",
        borderRadius: "15px",
        color: "blue",
        fontSize: "10px"
      }
    }
  }
])

const bars3 = ref([
  {
    beginDate: "15.01.2025 08:30",
    endDate: "20.01.2025 16:45",
    ganttBarConfig: {
      id: "9876543210",
      label: "Updated Bundle",
      bundle: "bundle3",
      style: {
        background: "cyan"
      }
    }
  },
  {
    beginDate: "20.02.2024 12:00",
    endDate: "10.03.2024 18:30",
    ganttBarConfig: {
      id: "1234567890",
      label: "New Task",
      hasHandles: true,
      style: {
        background: "#f79466"
      }
    }
  },
  {
    beginDate: "25.04.2024 09:15",
    endDate: "30.04.2024 21:00",
    ganttBarConfig: {
      id: "2468135790",
      label: "Greetings",
      style: {
        background: "#aabbcc",
        borderRadius: "8px",
        color: "white",
        fontSize: "12px"
      }
    }
  }
])

const bars4 = [
  {
    beginDate: "10.01.2024 08:00",
    endDate: "15.03.2024 16:30",
    ganttBarConfig: {
      id: "9876543210",
      label: "Novo Pacote",
      bundle: "pacote3",
      style: {
        background: "pink"
      }
    }
  },
  {
    beginDate: "05.03.2024 10:00",
    endDate: "15.04.2024 22:15",
    ganttBarConfig: {
      id: "2468135790",
      label: "hello folks",
      style: {
        background: "#ffd700",
        borderRadius: "10px",
        color: "black",
        fontSize: "14px"
      }
    }
  }
]

const addBar = () => {
  if (bars1.value.some((bar) => bar.ganttBarConfig.id === "test1")) {
    return
  }
  const bar = {
    beginDate: dayjs().add(1, "day").hour(4).startOf("hour").format(format.value),
    endDate: dayjs().add(2, "day").hour(4).startOf("hour").format(format.value),
    ganttBarConfig: {
      id: "test1",
      hasHandles: true,
      label: "Hello!",
      style: {
        background: "#5484b7",
        borderRadius: "20px"
      }
    }
  }
  bars1.value.push(bar)
}

const deleteBar = () => {
  const idx = bars1.value.findIndex((b) => b.ganttBarConfig.id === "test1")
  if (idx !== -1) {
    bars1.value.splice(idx, 1)
  }
}

const onClickBar = (bar: GanttBarObject, e: MouseEvent, datetime?: string) => {
  // console.log("click-bar", bar, e, datetime)
}

const onMousedownBar = (bar: GanttBarObject, e: MouseEvent, datetime?: string) => {
  // console.log("mousedown-bar", bar, e, datetime)
}

const onMouseupBar = (bar: GanttBarObject, e: MouseEvent, datetime?: string) => {
  // console.log("mouseup-bar", bar, e, datetime)
}

const onMouseenterBar = (bar: GanttBarObject, e: MouseEvent) => {
  // console.log("mouseenter-bar", bar, e)
}

const onMouseleaveBar = (bar: GanttBarObject, e: MouseEvent) => {
  // console.log("mouseleave-bar", bar, e)
}

const onDragstartBar = (bar: GanttBarObject, e: MouseEvent) => {
  // console.log("dragstart-bar", bar, e)
}

const onDragBar = (bar: GanttBarObject, e: MouseEvent) => {
  bar.beginDate = formatDate(bar.beginDate);
  bar.endDate = formatDate(bar.endDate);
  // console.log("drag-bar", bar, e)
}

const onDragendBar = (
  bar: GanttBarObject,
  e: MouseEvent,
  movedBars?: Map<GanttBarObject, { oldStart: string; oldEnd: string }>
) => {
  // console.log("dragend-bar", bar, e, movedBars)
}

const onContextmenuBar = (bar: GanttBarObject, e: MouseEvent, datetime?: string) => {
  // console.log("contextmenu-bar", bar, e, datetime)
}
</script>

<style>
.gantt-chart {
  width:80%;
  position:absolute;
  right:0;
}
.data {
  width:20%;
  position:absolute;
  left:0;
  margin-top: 52px
}

table {
    border-collapse: collapse;
    width: 100%; /* Ensure table takes full width */
  }

  /* Reset cell padding and margin */
  td, th {
    padding: 0;
    margin: 0;
  }

  /* Apply styles to table cells */
  .title {
    width: 70%;
    height: 28px;
    border: 1px solid red;
    box-sizing: border-box; /* Include border in width calculation */
  }

  .manager {
    width: 15%;
    height: 25px;
    border: 1px solid red;
    box-sizing: border-box; /* Include border in width calculation */
  }

  .label {
    width: 15%;
    height: 25px;
    border: 1px solid red;
    box-sizing: border-box; /* Include border in width calculation */
  }
</style>