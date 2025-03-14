<script lang="ts">
	import { initial_data } from './data';
	import { db } from './db';

	let form_data = $state(structuredClone(initial_data));
	let status: 'LOADING' | 'READY' = $state('LOADING');
	let loaded_id: number | undefined | null = $state(null);

	async function save() {
		const new_invoice_id = await db.invoices.add({
			date: new Date().toISOString(),
			form_data: $state.snapshot(form_data)
		});

		loaded_id = new_invoice_id;
		status = 'READY';
	}
</script>

<section>
	<h1>Invoice</h1>

	<div>
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
