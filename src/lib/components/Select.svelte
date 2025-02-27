<script lang="ts" context="module">
    export type SelectOption<T = unknown> = {
        value: T;
        label: string;
        icon?: string;
        group?: string;
    };
</script>

<script lang="ts">
    import { createSelect, melt, type CreateSelectProps } from '@melt-ui/svelte';
    import { createEventDispatcher } from 'svelte';
    import { fly, type FlyParams } from 'svelte/transition';

    export let options: Array<SelectOption>;
    export let nativeMobile = false;
    export let value: unknown | undefined = undefined;
    export let onSelectedChange: CreateSelectProps['onSelectedChange'] = undefined;
    // TODO: This id currently gets overriden by Melt. We should either use the label el, or
    // allow passing down ids in Melt.
    export let id: string | undefined = undefined;
    export let preventScroll = false;
    export let placement: NonNullable<CreateSelectProps['positioning']>['placement'] = 'bottom';

    const dispatch = createEventDispatcher<{
        change: unknown;
    }>();

    const {
        elements: { trigger, menu, option: optionEl, group: groupEl, groupLabel },
        states: { open, selected, selectedLabel }
    } = createSelect<unknown>({
        preventScroll,
        positioning: {
            sameWidth: true,
            fitViewport: true,
            placement
        },
        forceVisible: true,
        onSelectedChange({ curr, next }) {
            if (onSelectedChange) {
                onSelectedChange({ curr, next });
            }
            value = next?.value;
            dispatch('change', next?.value);

            return next;
        }
    });

    $: selectedOption = options.find((o) => o.value === value);

    $: if (selectedOption) {
        selected.set(selectedOption);
    }

    const DEFAULT_GROUP = 'default';
    type Group = {
        label: string;
        options: SelectOption<unknown>[];
    };
    $: groups = (function getGroups(): Group[] {
        const groups = options.reduce<Record<string, SelectOption[]>>((carry, option) => {
            const group = option.group ?? DEFAULT_GROUP;
            if (!carry[group]) {
                carry[group] = [];
            }
            carry[group].push(option);
            return carry;
        }, {});

        return Object.entries(groups).map(([label, options]) => ({ label, options }));
    })();

    $: flyParams = {
        duration: 150,
        y: placement === 'top' ? 4 : -4
    } as FlyParams;
</script>

<button
    class="aw-select is-colored"
    {id}
    class:aw-is-not-mobile={nativeMobile}
    use:melt={$trigger}
    aria-label="Select theme"
>
    <div class="physical-select">
        {#if selectedOption?.icon}
            <span class={selectedOption.icon} aria-hidden="true" />
        {/if}
        <span>{$selectedLabel}</span>
    </div>
    <span class="icon-cheveron-down" aria-hidden="true" />
</button>

{#if $open}
    <div
        class="aw-select-menu"
        class:aw-is-not-mobile={nativeMobile}
        style:z-index={10000}
        use:melt={$menu}
        transition:fly={flyParams}
    >
        {#each groups as group}
            {@const isDefault = group.label === DEFAULT_GROUP}
            {#if isDefault}
                <div class="u-flex u-flex-vertical u-gap-2">
                    {#each group.options as option}
                        <button class="aw-select-option" use:melt={$optionEl(option)}>
                            {#if option.icon}
                                <span class={option.icon} aria-hidden="true" />
                            {/if}
                            <span style:text-transform="capitalize">{option.label}</span>
                        </button>
                    {/each}
                </div>
            {:else}
                <div class="aw-select-group" use:melt={$groupEl(group.label)}>
                    <span class="aw-select-group-label" use:melt={$groupLabel(group.label)}>
                        {group.label}
                    </span>

                    {#each group.options as option}
                        <button class="aw-select-option" use:melt={$optionEl(option)}>
                            {#if option.icon}
                                <span class={option.icon} aria-hidden="true" />
                            {/if}
                            <span style:text-transform="capitalize">{option.label}</span>
                        </button>
                    {/each}
                </div>
            {/if}
        {/each}
    </div>
{/if}

<div
    class="aw-select is-colored aw-is-only-mobile"
    style:display={nativeMobile ? undefined : 'none'}
>
    {#if selectedOption?.icon}
        <span class={selectedOption.icon} aria-hidden="true" />
    {/if}
    <select {id} bind:value>
        {#each groups as group}
            {@const isDefault = group.label === DEFAULT_GROUP}
            {#if isDefault}
                {#each options as option}
                    <option value={option.value} selected={option.value === value}>
                        {option.label}
                    </option>
                {/each}
            {:else}
                <optgroup label={isDefault ? undefined : group.label}>
                    {#each options as option}
                        <option value={option.value} selected={option.value === value}>
                            {option.label}
                        </option>
                    {/each}
                </optgroup>
            {/if}
        {/each}
    </select>
    <span class="icon-cheveron-down" aria-hidden="true" />
</div>

<style lang="scss">
    .aw-select {
        min-width: var(--min-width, var(--p-select-min-width));
    }
</style>
