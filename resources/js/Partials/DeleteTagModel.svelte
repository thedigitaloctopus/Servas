<script>
    import Modal from "@/Components/Modals/Modal.svelte";
    import Button from "@/Components/Buttons/Button.svelte";
    import {Inertia} from '@inertiajs/inertia';
    import {dispatchCustomEvent, route} from '@/utils';
    import {refreshTags} from "@/stores";

    let showModal = false;
    let tag = {};

    function prepareDeleteTag(e) {
        tag = e.detail;

        showModal = true;
    }

    function deleteTag() {
        Inertia.delete(route('tags.destroy', tag.id), {
            preserveScroll: true,
            onSuccess: () => {
                // End the edit mode
                dispatchCustomEvent('tagDeletionWasSuccessful');
                showModal = false;
                $refreshTags = true;
            },
        });
    }
</script>

<svelte:window on:deleteTag={prepareDeleteTag}/>

<Modal title="Delete Tag" bind:showModal>
    <p class="text-sm text-gray-500">
        Are you sure you want to delete this tag?
    </p>
    <div class="mt-1 text-sm text-gray-700 font-medium">
        {tag.name}
    </div>

    <svelte:fragment slot="footer">
        <Button on:clicked={deleteTag} title="Delete" color="red" focusButton={true}
                class="focus:ring-offset-gray-50 sm:ml-3"/>
        <Button on:clicked={() => showModal = false} title="Cancel" color="white"
                class="hidden mt-3 focus:ring-offset-gray-50 sm:inline-flex sm:mt-0"/>
    </svelte:fragment>
</Modal>
