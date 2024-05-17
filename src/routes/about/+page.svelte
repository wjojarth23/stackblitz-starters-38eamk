<script>
  import { Button } from "$lib/components/ui/button";
  import * as Card from "$lib/components/ui/card";
  import * as Dialog from "$lib/components/ui/dialog";
  import * as Popover from "$lib/components/ui/popover";
  import { getLocalTimeZone, today } from "@internationalized/date";
  import { ScrollArea } from "$lib/components/ui/scroll-area/index.js";
  import { Separator } from "$lib/components/ui/separator/index.js";
  import { Calendar } from "$lib/components/ui/calendar/index.js";
  import * as Command from "$lib/components/ui/command";
  import PocketBase from "pocketbase";
  const pb = new PocketBase("https://flexapp.pockethost.io");
  import { onMount, onDestroy } from "svelte";
  let nextWeek = [];
  let registeredFlexes = [];
  let availableFlexes = [];
  const days = [
    "Sunday",
    "Monday",
    "Tuesday",
    "Wednesday",
    "Thursday",
    "Friday",
    "Saturday",
  ];
  let userId = "uk1btlafd64qiqc";
  let todayDate = new Date();
  function findObjectBySameDay(dateObject, objectArray) {
    // Loop through each object in the array
    for (const obj of objectArray) {
      // Check if the object has a "date" attribute
      if (obj.flex.hasOwnProperty("date")) {
        // Create a new Date object from the object's "date" attribute
        const objDate = new Date(obj.flex.date);

        // Check if year, month, and day match
        if (
          dateObject.getFullYear() === objDate.getFullYear() &&
          dateObject.getMonth() === objDate.getMonth() &&
          dateObject.getDate() === objDate.getDate()
        ) {
          // Return the matching object
          return obj;
        }
      }
    }

    // If no matching object found, return null
    return null;
  }
  console.log(nextWeek);
  onMount(async () => {
    const resultList = await pb
      .collection("active_flexes")
      .getList(1, 100000, { expand: "referrals,signups,activity" });
    console.log(resultList);
    for (let flexactivity of resultList.items) {
      console.log("uk1btlafd64qiqc" in flexactivity.referrals);
      if (flexactivity.referrals.includes(userId)) {
        console.log("refferd");
        registeredFlexes = [
          ...registeredFlexes,
          { type: "refferal", flex: flexactivity },
        ];
      }
      if (flexactivity.singnups.includes(userId)) {
        registeredFlexes = [
          ...registeredFlexes,
          { type: "signup", flex: flexactivity },
        ];
      }
    }
    console.log(registeredFlexes);
    for (let i = 0; i < 7; i++) {
      const nextDay = new Date();
      nextDay.setDate(todayDate.getDate() + i);

      const dayName = days[nextDay.getDay()];
      const formattedDate = nextDay.toLocaleDateString("en-US");
      nextWeek = [
        ...nextWeek,
        {
          day: dayName,
          date: formattedDate,
          registeredActivity: findObjectBySameDay(nextDay, registeredFlexes),
        },
      ];

      console.log(nextDay);
    }
  });
</script>

<div class="p-4 flex flex-cols w-full">
  <div class="border-l border-neutral-400 p-2"></div>
  {#each nextWeek as day, index}
    <Dialog.Root>
      <div class="border-r border-neutral-400 p-2 w-1/6">
        <div class="font-bold text-md font-mono uppercase">{day.day}</div>
        {day.date}
        <div class="h-8"></div>
        {#if day.registeredActivity?.flex.expand?.activity.name}
          {day.registeredActivity?.flex.expand?.activity.name}
        {:else}
          <div class="">Remember to Sign Up!</div>
        {/if}
        {#if day?.registeredActivity?.type != "refferal"}
          <Dialog.Trigger
            class="w-full bg-primary text-primary-foreground hover:bg-primary/90 p-1 w-full rounded-md"
            >Sign Up</Dialog.Trigger
          >
        {:else}<div>You have been referred</div>
        {/if}
      </div>
      <Dialog.Content>
        <Dialog.Header>
          <Dialog.Title>Schedule Flex Actity</Dialog.Title>
          <Dialog.Description>
            This action cannot be undone. This will permanently delete your
            account and remove your data from our servers.
          </Dialog.Description>
        </Dialog.Header>
      </Dialog.Content>
    </Dialog.Root>
  {/each}
</div>
