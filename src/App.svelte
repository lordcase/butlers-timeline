<script>
	import { quintOut } from 'svelte/easing';
	import { fade } from 'svelte/transition';
	import { crossfade } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import { tick } from 'svelte';


	const [send, receive] = crossfade({
		duration: d => Math.sqrt(d * 200),

		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});


	function arrangeElementsInCircle (elements, x, y, r) {
		if (elements.length > 1) {
			for (var i = 0; i < elements.length; i++) {
				// elements[i].scaleX = 1 / elements.length
				// elements[i].scaleY = 1 / elements.length
				elements[i].style.top = (x + r * Math.cos((2 * Math.PI) * i/elements.length)) + "px"
				elements[i].style.left = (y + r * Math.sin((2 * Math.PI) * i/elements.length)) + "px"
				console.log(elements[i].style.top,elements[i].style.left)
			}
		} else {
			elements[0].style.top = (x) + "px"
			elements[0].style.left = (y) + "px"
		}
	}
	function handleLoaded() {
		let shopContainers = document.getElementsByClassName("shop")
		arrangeElementsInCircle(shopContainers, 300, 300, 200)
	}
	
	function addShop() {
		employees.push(
			{id:employees.length+1, name:"New Shop"+employees.length, active:true}
		)
	}

	async function removeShop(shop) {
		shops = shops.filter(s => s !== shop);
		await tick()
		let shopContainers = document.getElementsByClassName("shop")
		setTimeout(()=>arrangeElementsInCircle(shopContainers, 300, 300, 200), 700)
	}
	
	function remove(employee) {
		employees = employees.filter(e => e !== employee);
	}

	function move(employee, shopId) {
		employee.shop = shopId;
		remove(employee);
		employees = employees.concat(employee);
		console.log(employees)
	}
	let eid = 1;
	let sid = 1;
	
	let employees = [
		{id: eid, name: 'Dobó Krisz', pos: '', pic: '', shop: 1, active: true},
		{id: 2, name: 'Dobó Krisz2', pos: '', pic: '', shop: 2, active: true},
		{id: 3, name: 'Dobó Krisz3', pos: '', pic: '', shop: 3, active: true},
	]
	let shops = [
		{id: 1, name: 'Árkád', active: true},
		{id: 2, name: 'Mom', active: true},
		{id: 3, name: 'Mammut', active: true},
		{id: 4, name: 'Mammut', active: true},
		{id: 5, name: 'Mammut', active: true},
	]
</script>
<style>
	#a {
		position: absolute;
		top: -100px;
		left: -100px;

	}
	.boardBack {
		display: inline-block;
		position: absolute;
		top: 100px;
		left: 100px;
		background-color: rgba(0,0,0,0.2);
		width: 400px;
		height: 400px;
		border-radius: 200px;
	}
	.board {
		display: block;
		position: relative;
		width: 100%;
		height: 100%;
	}
	.shop {
		border: 1px solid black;
		margin: 0 0 30px 0;
		position: absolute;
		top: 0;
		left: 0
	}
	.employee {
		border: 1px solid black;
		display: inline-block;
	}
</style>

<svelte:window on:load={handleLoaded}/>
<button on:click={addShop}>Add shop</button>
<button on:click={removeShop}>Remove shop</button>

<div class='board'>
	<div class='boardBack'></div>
	{#each shops.filter(s => s.active) as shop (shop.id)}
		<div class="shop" transition:fade>{shop.name}
			<a on:click="{() => removeShop(shop)}">-</a>
			{#each employees.filter(e => e.active && e.shop === shop.id) as e (e.id)}
				<div class="employee"
					in:receive="{{key: e.id}}"
					out:send="{{key: e.id}}"
					animate:flip
				>{e.name}
					<a on:click="{() => move(e, 1)}">1</a>
					<a on:click="{() => move(e, 2)}">2</a>
					<a on:click="{() => move(e, 3)}">3</a>
					<a on:click="{() => remove(e)}">-</a>
				</div>
			{/each}
		</div>
	{/each}
</div>