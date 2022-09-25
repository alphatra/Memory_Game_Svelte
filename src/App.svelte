<script lang="ts">
  let name: string
  enum STATE {
      NEW,
      RUNNING,
      PAUSED,
  }
  
  let state: STATE = STATE.NEW;
  let startTime: number = 0;
  let elaspedTime: number = 0;
  let oldElapsedTime: number = 0;
  let interval: any;
  let gameStatus: Boolean = false;
  let randomColor = () => Math.floor(Math.random() * 167727215).toString(16);
  
  const pad2 = (number: number) => `00${number}`.slice(-2);
  const pad3 = (number: number) => `00${number}`.slice(-2);
  let Game:{
      id: number;
      status: boolean;
  }
  let Squares_list: {
      value: string;status: boolean;found: boolean
  } [] = [];
  let Rank: {
      id: number;
      color: string;
      place: number;
      time: any;
  } [] = [];
  let Board: {
      title: string;
      status: boolean;
  } [] = []
  Board = [{title:'3x4',status:true},{title:'4x4',status:false},{title:'4x5',status:false}]
  let addToList = () => {
      Rank = [...Rank, {id: Rank.length,
          color: randomColor(),
          place: Rank.length==0?1:Rank[Rank.length-1].place+1,
          time: formattedElaspedTime,
      }];
  }
  
  $: hours = pad2(Math.floor(elaspedTime / 1000 / 60 / 60) % 60);
  $: minutes = pad2(Math.floor(elaspedTime / 1000 / 60) % 60);
  $: seconds = pad2(Math.floor(elaspedTime / 1000) % 60);
  $: millis = pad3(elaspedTime % 1000);
  $: formattedElaspedTime = `${hours}:${minutes}:${seconds}.${millis}`;
  
  function add_elem(icons) {
      for (let i = 0; i < icons.length; i++)
          for (let x = 0; x <= 1; x++)
              Squares_list = [...Squares_list,{
                  value: icons[i],
                  status: false,
                  found: false
              }]
              /*                Squares_list.push({
                  value: icons[i],
                  status: false,
                  found: false
              });*/
  }
  add_elem(["🎅", "❄️", "🎁", "🎄", "✨", "🛷"]);

  console.log(Squares_list);
  const Squares_listCopy = [...Squares_list]
  function shuffle(array): void {
      let currentIndex = array.length,
          randomIndex;
  
      // While there remain elements to shuffle.
      while (currentIndex != 0) {
  
          // Pick a remaining element.
          randomIndex = Math.floor(Math.random() * currentIndex);
          currentIndex--;
  
          // And swap it with the current element.
          [array[currentIndex], array[randomIndex]] = [
              array[randomIndex], array[currentIndex]
          ];
      }
  
      return array;
  }
  const start = () => {
      startTime = Date.now();
      state = STATE.RUNNING;
      interval = setInterval(() => {
          if (state === STATE.RUNNING) {
              const endTime = Date.now();
              elaspedTime = endTime - startTime + oldElapsedTime;
          }
      });
  };
  const place = (key) => {
      switch (key) {
          case 1:
              return '🏆'
          case 2:
              return '😎'
          case 3:
              return '💩'
          default:
            return '💩'
      }
      return ''
  }
  shuffle(Squares_list)
  
  $: if(Squares_list.every(item=>item.found == true)){
      const reset = () => {
          elaspedTime = 0;
          state = STATE.NEW;
          clearInterval(interval);
      };
      addToList();
      gameStatus = !gameStatus
      Squares_list.filter(item=>{item.found=false,item.status = false})
      shuffle(Squares_list);
      reset()
  };
  const handleClick = (i) => {
      if (!gameStatus) {
          start()
          gameStatus = !gameStatus
      }
      Squares_list[i].status = !Squares_list[i].status
      let items_active = Squares_list.filter(item => item.status == true);
      if (items_active.length === 2 && items_active[0].value === items_active[1].value) {
          items_active.forEach(element => element.found = true)
      }
      if (items_active.length > 2) {
          Squares_list[i].status = !Squares_list[i].status
          items_active.filter(item => {
              item.status = item.status ? false : true
          });
      }
  }
  const changeBoard = (i) => {
      Board.forEach(element => {
          element.status=false
      });
      Board[i].status= true
      switch (i) {
          case 0:
              Squares_list = Squares_listCopy;
              shuffle(Squares_list);
              break;
          case 1:
              Squares_list = Squares_listCopy;
              add_elem(["☃️", "🔔"]);
              shuffle(Squares_list);
              break;
          case 2:
              Squares_list = Squares_listCopy;
              add_elem(["☃️", "🔔","🦌","🧝"]);
              shuffle(Squares_list);
              break;
      }
  }
    console.log(Rank);
    
  </script>
<div class="grid h-screen place-items-center">
    <div class="grid overflow-hidden auto-cols-auto auto-rows-auto gap-2 grid-flow-row">
        <div class="min-w-full min-h-full row-span-1 col-start-1 sm:col-span-2 ">
            <p class="text-2xl sm:text-2xl md:text-4xl mb-2">Ustaw plansze:</p>
            {#each Board as button,id}
            <button class="{button.status?'btn btn-active btn-primary btn-lg md:btn-lg sm:btn-sm mx-1':'btn btn-outline btn-lg btn-primary md:btn-lg sm:btn-sm mx-1'}" on:click={()=>changeBoard(id)}>{button.title}</button>
            {/each}
        </div>
        <div class="min-w-full min-h-full row-span-3 bg-red-500 rounded">
            <div class="{Board[0].status?'grid grid-cols-3 gap-3 m-4':'grid grid-cols-4 gap-3 m-4'}" >
            {#each Squares_list as tile, id}
            <button class="sm:text-2xl md:text-4xl content-center text-center md:w-32 md:h-32 {tile.found?'solved':''} {Board[0].status?'w-24 h-24':'w-16 h-16'} bg-zinc-900" on:click={()=>handleClick(id)}>{tile.status || tile.found?tile.value:'?'}</button>
            {/each}
            </div>
        </div>
        <div class="bg-red-500 text-2xl sm:text-2xl md:text-4xl w-64 font-semibold p-2 px-7 grid row-span-1 rounded place-items-center place-content-center">
            <p>{formattedElaspedTime}</p>
        </div>
        <div class="overflow-x-auto">
            <table class="table w-full">
              <tbody>
                {#each Rank as player}
                <tr>
                    <th style="background-color:#{player.color}">{player.place}</th>
                    <td style="background-color:#{player.color}">{place(player.place)}</td>
                    <td style="background-color:#{player.color}">{player.time}</td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
    </div>
</div>
<style global lang="postcss">
  @tailwind base;
  @tailwind components;
  @tailwind utilities;
  @keyframes example {
    from {background-color: rgb(255, 255, 255);}
      to {background-color: rgb(40, 51, 206);}
  }
  .solved{
    background-color: blue;
    animation-name: example;
    animation-duration: 1s;
  }
</style>