<script setup lang="ts">
import type { Column, Task } from "~/types";
import draggable from "vuedraggable";
import { useKeyModifier } from "@vueuse/core";
import { nanoid } from "nanoid";

// ------ useLocalStorage is pretty much a drop-in replacement for ref
// ------ but it also syncs all the data that was in the ref before
const columns = useLocalStorage<Column[]>("trelloBoard", [
    {
        id: nanoid(),
        title: "Backlog",
        tasks: [
            {
                id: nanoid(),
                title: "Create marketing landing page",
                createdAt: new Date(),
            },
            {
                id: nanoid(),
                title: "Develop cool new feature",
                createdAt: new Date(),
            },
            {
                id: nanoid(),
                title: "Fix page nav bug",
                createdAt: new Date(),
            },
        ],
    },

    { title: "Selected for Dev", id: nanoid(), tasks: [] },
    { title: "In progress", id: nanoid(), tasks: [] },
    { title: "QA", id: nanoid(), tasks: [] },
    { title: "Complete", id: nanoid(), tasks: [] },
]);
const alt = useKeyModifier("Alt");

watch(columns, () => {
    // ajax request
}, {
    deep: true,
})

function createColumn() {
    const column: Column = {
        id: nanoid(),
        title: "",
        tasks: [],
    }
    columns.value.push(column);
    nextTick(() => {
        (document.querySelector(".column:last-of-type .title-input") as HTMLInputElement).focus()
    });
}

</script>

<template>
    <div class="flex overflow-x-auto items-start gap-4">
        <draggable v-model="columns" group="columns" :animation="200" handle=".drag-handle" item-key="id"
            class="flex gap-4 items-start">
            <template #item="{ element: column }: { element: Column }">

                <div class="column bg-gray-300 p-5 rounded min-w-[250px]">
                    <header class="font-bold mb-4">
                        <!-- <span class="drag-handle cursor-move" > :: </span> -->
                        <DragHandle />
                        <!-- {{ column.title }} -->
                        <input class="title-input bg-transparent focus:bg-white rounded px-1 w-4/5"
                            @keyup.enter="($event.target as HTMLInputElement).blur()" @keydown.backspace="column.title == ''
                                ? (columns = columns.filter((c) => c.id !== column.id))
                                : null" type="text" v-model="column.title" />
                    </header>

                    <draggable v-model="column.tasks" :group="{ name: 'tasks', pull: alt ? 'clone' : true }"
                        handle=".drag-handle" :animation="150" item-key="id">

                        <template #item="{ element: task } : { element: Task }">

                            <div>
                                <TrelloBoardTask :task="task"
                                    @delete="column.tasks = column.tasks.filter((t) => t.id !== $event)" />
                            </div>
                        </template>

                    </draggable>


                    <footer>
                        <!-- <button class="text-gray-500"> + Add a Card </button> -->
                        <NewTask @add="column.tasks.push($event)" />
                    </footer>
                </div>
            </template>
        </draggable>

        <button @click="createColumn" class="bg-gray-300 whitespace-nowrap p-2 rounded opacity-50">
            + Add Another Column
        </button>

    </div>


    <!-- for printing test data -->
    <!-- <pre>
    {{ columns }}
</pre> -->

</template>
