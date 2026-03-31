<script lang="ts">
  import { toast } from "svelte-sonner";
  import type { AgentConfigs, AgentDefinition } from "tauri-plugin-modular-agent-api";

  import * as Card from "$lib/components/ui/card/index.js";
  import { FieldGroup, Field, FieldLabel } from "$lib/components/ui/field/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import { Switch } from "$lib/components/ui/switch/index.js";
  import { setGlobalConfigs } from "$lib/modular_agent";

  import FieldDescription from "@/lib/components/ui/field/field-description.svelte";

  interface Props {
    agentName: string;
    agentConfigs: AgentConfigs;
    agentDef: AgentDefinition | null;
  }

  const { agentName, agentConfigs, agentDef }: Props = $props();

  // configs is $derived — bind:value mutates the underlying agentConfigs object in-place.
  // Do not change this to clone/spread, or auto-save will read stale data.
  let configs = $derived.by(() => {
    let ac = agentConfigs;
    return ac;
  });
  let ad = $derived.by(() => {
    let ad = agentDef;
    return ad;
  });

  async function autoSaveConfig() {
    try {
      await setGlobalConfigs(agentName, configs);
    } catch (e) {
      toast.error("Failed to save agent settings");
      console.error("Failed to save agent settings:", e);
    }
  }

  function blurOnEnter(e: KeyboardEvent) {
    if (e.key === "Enter") (e.target as HTMLInputElement).blur();
  }
</script>

<Card.Root class="@container/card">
  <Card.Header>
    <Card.Title>{ad?.title ?? agentName}</Card.Title>
    <Card.Description>{ad?.description}</Card.Description>
  </Card.Header>
  <Card.Content>
    {#if ad?.global_configs}
      <FieldGroup>
        {#each Object.entries(ad.global_configs) as [key, globalConfig]}
          <Field orientation="horizontal" class="grid gap-4 sm:grid-cols-[220px_1fr] items-center">
            <div class="flex flex-col">
              <FieldLabel>
                {globalConfig?.title || key}
              </FieldLabel>
              <FieldDescription>{globalConfig?.description}</FieldDescription>
            </div>
            {@const ty = globalConfig.type}
            {#if ty === "boolean"}
              <Switch bind:checked={configs[key]} onCheckedChange={() => autoSaveConfig()} />
            {:else if ty === "integer"}
              <Input
                bind:value={configs[key]}
                onchange={() => autoSaveConfig()}
                onkeydown={blurOnEnter}
              />
            {:else if ty === "number"}
              <Input
                type="number"
                bind:value={configs[key]}
                onchange={() => autoSaveConfig()}
                onkeydown={blurOnEnter}
              />
            {:else if ty === "string"}
              <Input
                type="text"
                bind:value={configs[key]}
                onchange={() => autoSaveConfig()}
                onkeydown={blurOnEnter}
              />
            {:else if ty === "text"}
              <Input
                bind:value={configs[key]}
                onchange={() => autoSaveConfig()}
                onkeydown={blurOnEnter}
              />
            {:else if ty === "password"}
              <Input
                type="password"
                bind:value={configs[key]}
                onchange={() => autoSaveConfig()}
                onkeydown={blurOnEnter}
              />
            {:else if ty === "object"}
              <Input
                bind:value={configs[key]}
                onchange={() => autoSaveConfig()}
                onkeydown={blurOnEnter}
              />
            {:else}
              <Input type="text" value={JSON.stringify(configs[key], null, 2)} disabled />
            {/if}
          </Field>
        {/each}
      </FieldGroup>
    {/if}
  </Card.Content>
</Card.Root>
