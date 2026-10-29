<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem the reports in `ticket` really hold. causes.ts places a report by tenant and
  // channel; the reports the desk read as a one-day move (desk-notes.json) are a problem of their own.
  type Problem = 'parser' | 'hint' | 'one-day move' | 'unplaced';
  let { ticket, filed = {} }: { ticket: number; filed?: Partial<Record<Problem, string>> } = $props();

  const rows = $derived((reports as Report[]).filter((r) => r.mergedInto === ticket));
  const merged = $derived(uniqueById(rows));
  const shifted = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const problems = $derived([
    ...groupByCause(merged.filter((r) => !notes.oneDayMove.includes(r.id)), tenantOf).map((g) => ({
      problem: g.cause as Problem,
      reports: g.reports,
    })),
    { problem: 'one-day move' as Problem, reports: shifted },
  ].filter((p) => p.reports.length));
  const tenants = (rs: Report[]) => new Set(rs.map((r) => r.tenant)).size;
  const seen = (rs: Report[]) => {
    const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
    return `${days[0]} → ${days[days.length - 1]}`;
  };
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports in #{ticket} ({rows.length - merged.length} rows in the export are
  copies of a report) · <strong>{problems.length}</strong> problems
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each problems as p (p.problem)}
      <tr>
        <td><code>{p.problem}</code></td>
        <td>{p.reports.length}</td>
        <td>{tenants(p.reports)}</td>
        <td>{seen(p.reports)}</td>
        <td>{#if filed[p.problem]}<a href={filed[p.problem]}>{filed[p.problem]?.split('/').slice(-3).join('/').replace('/issues/', '#')}</a>{:else}—{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
</style>
