<script context="module">
    import AppLayout, {title} from "@/Layouts/AppLayout/AppLayout.svelte";

    export const layout = AppLayout;
</script>

<script>
    import DeleteUserForm from '@/Pages/Profile/Partials/DeleteUserForm.svelte'
    import JetSectionBorder from '@/Jetstream/SectionBorder.svelte'
    import LogoutOtherBrowserSessionsForm from '@/Pages/Profile/Partials/LogoutOtherBrowserSessionsForm.svelte'
    import TwoFactorAuthenticationForm from '@/Pages/Profile/Partials/TwoFactorAuthenticationForm.svelte'
    import UpdatePasswordForm from '@/Pages/Profile/Partials/UpdatePasswordForm.svelte'
    import UpdateProfileInformationForm from '@/Pages/Profile/Partials/UpdateProfileInformationForm.svelte'
    import Main from "@/Layouts/AppLayout/Partials/Main.svelte";
    import {page} from "@inertiajs/inertia-svelte";

    export let sessions;

    $title = 'Profile Settings';
</script>

<Main>
    <div>
        <div class="max-w-7xl mx-auto py-10 space-y-14 sm:px-6 sm:space-y-0 lg:px-8">
            {#if $page.props.jetstream.canUpdateProfileInformation}
                <UpdateProfileInformationForm user={$page.props.user}/>

                <JetSectionBorder/>
            {/if}


            {#if $page.props.jetstream.canUpdatePassword}
                <UpdatePasswordForm class="mt-10 sm:mt-0"/>

                <JetSectionBorder/>
            {/if}

            {#if $page.props.jetstream.canManageTwoFactorAuthentication}
                <TwoFactorAuthenticationForm class="mt-10 sm:mt-0"/>

                <JetSectionBorder/>
            {/if}

            <LogoutOtherBrowserSessionsForm {sessions} class="mt-10 sm:mt-0"/>

            {#if $page.props.jetstream.hasAccountDeletionFeatures}
                <JetSectionBorder/>

                <DeleteUserForm class="mt-10 sm:mt-0"/>
            {/if}
        </div>
    </div>
</Main>
