<script lang="ts">
    import { invalidate } from '$app/navigation';
    import { page } from '$app/stores';
    import { Submit, trackError, trackEvent } from '$lib/actions/analytics';
    import { CardGrid, Heading } from '$lib/components';
    import { Dependencies } from '$lib/constants';
    import { Button, Form, FormList, InputNumber } from '$lib/elements/forms';
    import { addNotification } from '$lib/stores/notifications';
    import { sdk } from '$lib/stores/sdk';
    import { onMount } from 'svelte';
    import { func } from '../store';

    const functionId = $page.params.function;
    let timeout: number = null;

    onMount(async () => {
        timeout ??= $func.timeout;
    });

    async function updateTimeout() {
        try {
            await sdk.forProject.functions.update(
                functionId,
                $func.name,
                $func.execute || undefined,
                $func.events || undefined,
                $func.schedule || undefined,
                timeout,
                $func.enabled
            );
            await invalidate(Dependencies.FUNCTION);
            addNotification({
                type: 'success',
                message: 'Timeout has been updated'
            });
            trackEvent(Submit.FunctionUpdateTimeout);
        } catch (error) {
            addNotification({
                type: 'error',
                message: error.message
            });
            trackError(error, Submit.FunctionUpdateTimeout);
        }
    }
</script>

<Form onSubmit={updateTimeout}>
    <CardGrid>
        <Heading tag="h6" size="7">Timeout</Heading>
        <p>Limit the execution time of your function. Maximum value is 900 seconds (15 minutes).</p>
        <svelte:fragment slot="aside">
            <FormList>
                <InputNumber
                    min={1}
                    max={900}
                    id="time"
                    label="Time (in seconds)"
                    bind:value={timeout} />
            </FormList>
        </svelte:fragment>

        <svelte:fragment slot="actions">
            <Button disabled={$func.timeout === timeout || timeout < 1} submit>Update</Button>
        </svelte:fragment>
    </CardGrid>
</Form>
