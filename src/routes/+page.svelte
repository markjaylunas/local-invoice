<script lang="ts">
	import { initial_data } from './data';
	import type { Invoice } from './db';
	import { db } from './db';

	let form_data = $state(structuredClone(initial_data));
	let status: 'LOADING' | 'READY' = $state('LOADING');
	let loaded_id: number | undefined | null = $state(null);
	let all_invoices: Invoice[] = $state([]);

	async function save() {
		const new_invoice_id = await db.invoices.add({
			date: new Date().toISOString(),
			form_data: $state.snapshot(form_data)
		});

		loaded_id = new_invoice_id;
		status = 'READY';
		load_all_invoices();
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

	$effect(() => {
		load_all_invoices();
	});
</script>

<section>
	<h1>Invoice</h1>

	<div>
		<select onchange={load_invoice} name="all_invoices" id="all_invoices">
			{#each all_invoices as invoice}
				<option value={invoice.id}>{invoice.date}</option>
			{/each}
		</select>

		<button onclick={save}>Save</button>
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
