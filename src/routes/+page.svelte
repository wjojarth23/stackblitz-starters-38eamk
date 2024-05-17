<script>
import PocketBase from 'pocketbase';
import { Button } from "$lib/components/ui/button";
import * as Card from "$lib/components/ui/card";
import * as Dialog from "$lib/components/ui/dialog";
import * as Popover from "$lib/components/ui/popover";
import { getLocalTimeZone, today } from "@internationalized/date";
import { ScrollArea } from "$lib/components/ui/scroll-area/index.js";
import { Separator } from "$lib/components/ui/separator/index.js";
  import { Calendar } from "$lib/components/ui/calendar/index.js";
  import * as Command from "$lib/components/ui/command";
  let value = today(getLocalTimeZone());
const pb = new PocketBase('https://flexapp.pockethost.io');
let allActivities = [];
let allUsers =[];
let allFlexes = [];
let schedulingModal=false;
let unsubscribe= null;
let currentlyScheduling = false;
const tags = Array.from({ length: 10 }).map(
    (_, i, a) => `v1.2.0-beta.${a.length - i}`
  );
async function getAllActivities(){
let rawActivities = await pb.collection('activities').getList();

allActivities = rawActivities.items;
console.log(allActivities);
}
async function getAllUsers(){
let rawUsers = await pb.collection('users').getList();

allUsers = rawUsers.items;
}
async function getAllFlexes(){
  const collection = pb.collection("active_flexes");
      // Fetch initial data
      const records = await collection.getList(); // Adjust limit and offset as needed
      allFlexes = records.items // Update with items array


}
async function handlePost(activityDetails){
  console.log("hello")
  console.log(activityDetails)
  console.log(value.month,value.day,value.year)
  const newFlexPost = await pb.collection('active_flexes').create({
    activity:activityDetails.id,
    date:value
});
}
async function referrStudent(flex,student,index){
  pb.collection('active_flexes').update(flex.id, {
    referrals: [...flex.referrals,student.id]});
 const updatedFlexes = allFlexes.map((flex, idx) => {
    if (idx === index+1) {
      return { ...flex, referrals: [student.id,...flex.referrals] };
    }
    return flex;
  });

  allFlexes = updatedFlexes;
 console.log(allFlexes[index].refferals);
 }

async function getUserName(id){
  record = await pb.collection('users').getOne(id);
  return record
}
onMount(async () => {
getAllActivities();
getAllFlexes();
getAllUsers();
 unsubscribe = await pb
      .collection('active_flexes')
      .subscribe('*', async ({ action, record }) => {
        if (action === 'create') {
          allFlexes = [...allFlexes, record];
        }
        if (action === 'delete') {
          allFlexes = allFlexes.filter((m) => m.id !== record.id);
        }
      });
});
</script>

<div class="flex p-4 gap-2">
{#each allActivities as activity, index}
<Popover.Root>
<Card.Root>
<Card.Header>{activity.name}</Card.Header><Card.Content><div>{activity.maxStudents}</div><div>Max Students</div></Card.Content><Card.Footer><Popover.Trigger class = "bg-primary text-primary-foreground hover:bg-primary/90 p-1 w-full rounded-md">Post Session</Popover.Trigger></Card.Footer></Card.Root>
<br/>
<Popover.Content class="justify-center items-center w-80">
<Calendar bind:value/>
<Button on:click = {handlePost(activity)} >Create Flextime</Button>
  </Popover.Content>
</Popover.Root>
{/each}
<br/>
{#each allFlexes as flex, index}
<Popover.Root>
<Card.Root>
<Card.Header>{(allActivities.find(entry => entry.id === flex.activity)).name}</Card.Header>
<Card.Content>
<div class="flex gap-2 justify-center flex-col">
{flex.date.month}/{flex.date.day}/{flex.date.year}

<ScrollArea class="rounded-md p-1 border w-full h-20">
<div class = "p-2">
<h4 class="mb-4 text-sm font-medium leading-none">Referred Students</h4>
{#each flex.referrals as tag}
  <div class="text-sm">
    {(allUsers.find(entry => entry.id === tag)).name}
    <Separator class="my-2" />
  </div>
{/each}

</div>
</ScrollArea>
<Popover.Trigger class = "bg-primary text-primary-foreground hover:bg-primary/90 p-1 w-full rounded-md">Refer Students</Popover.Trigger>
<Popover.Content>
<Command.Root>
  <Command.Input placeholder="Type a command or search..." />
  <Command.List>
    <Command.Empty>No results found.</Command.Empty>
    {#each allUsers.filter(obj => obj.role === "student") as student , index}
      <Command.Item onSelect={(currentValue) =>{referrStudent(flex,student,index)}}>{student.name}</Command.Item>
    {/each}
  </Command.List>
</Command.Root>
</Popover.Content>
</div>
</Card.Content>
</Card.Root>

</Popover.Root>
{/each}
</div>