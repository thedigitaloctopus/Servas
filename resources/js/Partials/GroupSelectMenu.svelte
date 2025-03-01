<script>
    import Button from "@/Components/Buttons/Button.svelte";
    import Modal from '@/Components/Modals/Modal.svelte';
    import {isUndefined} from "lodash";

    export let selectedGroups = [];        // property to access from outside; contains group IDs; will only change when clicking "OK"

    let groups = [];
    let filteredGroups = [];
    let internalSelectedGroups = [];       // property for internal use only; contains group objects
    let showModal = false;
    let showSelectView = true;
    let modeIsMultiSelect = false;

    export const enableMultiSelectMode = () => modeIsMultiSelect = true;
    export const disableMultiSelectMode = () => modeIsMultiSelect = false;

    export async function openModal() {
        await getAllGroups();

        filterByGroup(null);

        showModal = true;
    }

    function saveChanges() {
        selectedGroups = [];

        internalSelectedGroups.forEach((group) => {
            selectedGroups = [...selectedGroups, group.id];
        });

        showModal = false;
    }

    async function getAllGroups() {
        await axios.get('/all-groups')
            .then(response => {
                groups = response.data
            });
    }

    function filterByGroup(groupId) {
        let filteredGroupsTemp = groups.filter((e) => {
            return e.parentGroupId === groupId;
        });

        if (filteredGroupsTemp.length > 0) {
            filteredGroups = filteredGroupsTemp;
        }
    }

    function selectGroup(group) {
        if (modeIsMultiSelect) {
            let index = getIndexOfGroupId(group.id, internalSelectedGroups);

            if (index !== -1) {
                internalSelectedGroups.splice(index, 1);
                internalSelectedGroups = internalSelectedGroups;
            } else {
                internalSelectedGroups = [...internalSelectedGroups, group];
            }
        } else {
            // The array can contain only one group when the mode is not "multi select".
            if (getIndexOfGroupId(group.id, internalSelectedGroups) !== -1) {
                internalSelectedGroups = [];
            } else {
                internalSelectedGroups = [group];
            }
        }
    }

    function goBack() {
        let parentGroup = groups.find(({id}) => {
            return id === filteredGroups[0]?.parentGroupId;
        })

        if (isUndefined(parentGroup)) {
            return;
        }

        filterByGroup(parentGroup.parentGroupId);
    }

    export function reset() {
        filteredGroups = [];
        selectedGroups = [];
        internalSelectedGroups = [];
        showSelectView = true;
    }

    export async function setSelectedGroups(groupsToSet) {
        if (!groupsToSet.length) {
            internalSelectedGroups = [];
            selectedGroups = [];

            return;
        }

        await getAllGroups();

        internalSelectedGroups = [];
        selectedGroups = [];

        if (modeIsMultiSelect) {
            groupsToSet.forEach((groupId) => {
                let group = groups.find(({id}) => {
                    return id === groupId;
                })

                if (!isUndefined(group)) {
                    internalSelectedGroups = [...internalSelectedGroups, group];
                }
            });

            selectedGroups = groupsToSet;
        } else {
            let group = groups.find(({id}) => {
                return id === groupsToSet[0];
            });

            if (!isUndefined(group)) {
                internalSelectedGroups = [group];
            }

            selectedGroups = groupsToSet[0] === null ? [] : [groupsToSet[0]];
        }
    }

    function getIndexOfGroupId(groupId, arrayToSearch) {
        return arrayToSearch.findIndex(group => group.id === groupId);
    }
</script>

<Modal title="Select a group" bind:showModal>

    <div class="border-b border-gray-200">
        <nav class="-mb-px flex space-x-8" aria-label="Tabs">
            <button on:click={() => showSelectView = true} type="button"
                    class={[showSelectView ? 'border-primary-500 text-primary-600' : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-200',
                    'w-1/2 flex justify-center whitespace-nowrap flex py-4 px-1 border-b-2 font-medium text-sm'].join(' ').trim()}>
                Select
            </button>

            <button on:click={() => showSelectView = false} type="button"
                    class={[!showSelectView ? 'border-primary-500 text-primary-600' : 'border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-200',
                    'w-1/2 flex justify-center whitespace-nowrap flex py-4 px-1 border-b-2 font-medium text-sm'].join(' ').trim()}>
                Selected

                <span
                    class={[!showSelectView ? 'bg-primary-100 text-primary-600' : 'bg-gray-100 text-gray-900',
                    'inline-block ml-3 py-0.5 px-2.5 rounded-full text-xs font-medium'].join(' ').trim()}>
                    {internalSelectedGroups.length}
                </span>
            </button>
        </nav>
    </div>

    <div class="mt-5">
        {#if showSelectView}
            <div class="flex gap-x-2">
                <Button on:clicked={goBack} color="white" class="!ring-0 md:!w-full">
                    <svg slot="icon" class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"
                         fill="currentColor">
                        <path fill-rule="evenodd"
                              d="M9.707 16.707a1 1 0 01-1.414 0l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l4.293 4.293a1 1 0 010 1.414z"
                              clip-rule="evenodd"/>
                    </svg>
                </Button>
                <Button on:clicked={() => filterByGroup(null)} color="white" class="!ring-0 md:!w-full">
                    <svg slot="icon" class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"
                         fill="currentColor">
                        <path
                            d="M10.707 2.293a1 1 0 00-1.414 0l-7 7a1 1 0 001.414 1.414L4 10.414V17a1 1 0 001 1h2a1 1 0 001-1v-2a1 1 0 011-1h2a1 1 0 011 1v2a1 1 0 001 1h2a1 1 0 001-1v-6.586l.293.293a1 1 0 001.414-1.414l-7-7z"/>
                    </svg>
                </Button>
            </div>

            <ul class="mt-3 divide-y divide-gray-100">
                {#each filteredGroups as group (group.id)}
                    <li on:click={() => filterByGroup(group.id)}
                        class="flex justify-between items-center text-gray-900 cursor-default select-none relative py-2.5 pl-8 pr-4">
                        <div class="font-normal block truncate">{group.title}</div>

                        {#if getIndexOfGroupId(group.id, internalSelectedGroups) !== -1}
                            <div class="text-indigo-600 absolute inset-y-0 left-0 flex items-center pl-1.5">
                                <!-- Heroicon name: solid/check -->
                                <svg class="h-5 w-5" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"
                                     fill="currentColor"
                                     aria-hidden="true">
                                    <path fill-rule="evenodd"
                                          d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                                          clip-rule="evenodd"/>
                                </svg>
                            </div>
                        {/if}
                        <button on:click|stopPropagation={() => selectGroup(group)} type="button"
                                class="text-sm text-gray-700">
                            {getIndexOfGroupId(group.id, internalSelectedGroups) !== -1 ? 'Unselect' : 'Select'}
                        </button>
                    </li>

                {:else}
                    <li class="py-2.5 px-4 text-gray-600 text-center">No groups found</li>
                {/each}
            </ul>
        {/if}

        {#if !showSelectView}
            <ul class="divide-y divide-gray-100">
                {#each internalSelectedGroups as group (group.id)}
                    <li class="flex justify-between items-center text-gray-900 cursor-default select-none relative py-2.5 px-4">
                        <div class="font-normal block truncate">{group.title}</div>
                        <button on:click|stopPropagation={() => selectGroup(group)} type="button"
                                class="text-sm text-gray-700">
                            Unselect
                        </button>
                    </li>

                {:else}
                    <li class="py-2.5 px-4 text-gray-600 text-center">No groups selected</li>
                {/each}
            </ul>
        {/if}
    </div>

    <svelte:fragment slot="footer">
        <Button on:clicked={saveChanges} title="OK"
                class="focus:ring-offset-gray-50 sm:ml-3"/>
        <Button on:clicked={() => showModal = false} title="Cancel" color="white"
                class="hidden mt-3 focus:ring-offset-gray-50 sm:inline-flex sm:mt-0"/>
    </svelte:fragment>
</Modal>
