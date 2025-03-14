<script lang="ts">
	import { initial_data } from './data';
	import type { Invoice } from './db';
	import { db } from './db';

	let form_data = $state(structuredClone(initial_data));
	let status: 'LOADING' | 'READY' = $state('LOADING');
	let loaded_id: number | undefined | null = $state(null);
	let all_invoices: Invoice[] = $state([]);

	async function save(data: typeof initial_data) {
		if (loaded_id) {
			await db.invoices.update(loaded_id, {
				form_data: data,
				date: new Date().toISOString()
			});
			return;
		} else {
			const new_invoice_id = await db.invoices.add({
				date: new Date().toISOString(),
				form_data: data
			});

			loaded_id = new_invoice_id;
			status = 'READY';
			load_all_invoices();
			load_recent_invoice();
		}
	}

	async function load_all_invoices() {
		all_invoices = await db.invoices.orderBy('date').toArray();
	}

	async function load_invoice(e: Event) {
		status = 'LOADING';
		const select = e.target as HTMLSelectElement;
		const id = select.value ? parseInt(select.value) : null;

		if (id) {
			const invoice = await db.invoices.get(id);

			if (invoice) {
				form_data = invoice.form_data;
				loaded_id = invoice.id;
			}
		}

		status = 'READY';
	}

	async function load_recent_invoice() {
		status = 'LOADING';
		const invoice = await db.invoices.orderBy('date').last();

		if (invoice) {
			form_data = invoice.form_data;
			loaded_id = invoice.id;
		}

		status = 'READY';
	}

	function new_invoice() {
		loaded_id = null;
		form_data = structuredClone(initial_data);
	}

	async function export_data() {
		try {
			const all_invoices = await db.invoices.orderBy('date').toArray();
			const json_string = JSON.stringify(all_invoices, null, 2);
			const blob = new Blob([json_string], { type: 'application/json' });
			const url = URL.createObjectURL(blob);
			const link = document.createElement('a');
			link.href = url;
			link.download = 'invoices.json';
			document.body.appendChild(link);
			link.click();
			document.body.removeChild(link);
			URL.revokeObjectURL(url);
		} catch (error) {
			console.error(error);
		}
	}

	$effect(() => {
		load_all_invoices();
		load_recent_invoice();
	});

	$effect(() => {
		if (status === 'READY') {
			save($state.snapshot(form_data));
		}
	});
</script>

<section>
	<h1>Invoice</h1>

	<div>
		<select value={loaded_id} onchange={load_invoice} name="all_invoices" id="all_invoices">
			{#each all_invoices as invoice}
				<option value={invoice.id}>{invoice.date}</option>
			{/each}
		</select>

		<button onclick={new_invoice}>New Invoice</button>
		<button onclick={() => save($state.snapshot(form_data))}>Save</button>
		<button onclick={export_data}>Export</button>
	</div>
	<div>
		<div>
			<label for="invoice">Invoice #</label>
			<input type="text" id="invoice" name="invoice" bind:value={form_data.invoice} />
		</div>
	</div>
	<div>
		<div>
			<label for="name">Name</label>
			<input type="text" id="name" name="name" bind:value={form_data.name} />
		</div>
	</div>
</section>
