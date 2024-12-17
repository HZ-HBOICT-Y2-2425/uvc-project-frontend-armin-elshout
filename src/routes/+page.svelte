<script>
    import "../app.css";
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    let activities = [];
    let newTitle = "";
    let newDate = "";
    let newLink = "";
    let editingActivity = null; // To track the activity being edited

    // Fetch all activities
    async function fetchData() {
        try {
            const response = await fetch("http://localhost:3011/api/activities");
            if (!response.ok) throw new Error("Failed to fetch activities");
            activities = await response.json();
        } catch (error) {
            alert("Could not load activities.");
            console.error(error);
        }
    };

    // Add a new activity
    const addActivity = async () => {
        if (newTitle.trim() && newDate.trim()) {
            const newActivity = { title: newTitle, date: newDate, link: newLink || "#" };
            try {
                const response = await fetch("http://localhost:3011/api/activities", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(newActivity),
                });
                if (!response.ok) throw new Error(`HTTP Error: ${response.status}`);
                const savedActivity = await response.json();
                activities = [...activities, savedActivity];
                newTitle = "";
                newDate = "";
                newLink = "";
            } catch (error) {
                alert("Failed to save activity.");
                console.error(error);
            }
        } else {
            alert("Please fill out both the title and date fields.");
        }
    };

    // Edit an activity
    const editActivity = async () => {
        if (!editingActivity || !editingActivity.title.trim() || !editingActivity.date.trim()) {
            alert("Please fill out both the title and date fields.");
            return;
        }
        try {
            const response = await fetch("http://localhost:3011/api/activities", {
                method: "PUT",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify(editingActivity),
            });
            if (!response.ok) throw new Error(`HTTP Error: ${response.status}`);
            const updatedActivity = await response.json();
            activities = activities.map((activity) =>
                activity.id === updatedActivity.id ? updatedActivity : activity
            );
            editingActivity = null;
        } catch (error) {
            alert("Failed to update activity.");
            console.error(error);
        }
    };

    // Delete an activity
    const deleteActivity = async (id) => {
        try {
            const response = await fetch(`http://localhost:3011/api/activities/${id}`, {
                method: "DELETE",
            });
            if (!response.ok) throw new Error(`HTTP Error: ${response.status}`);
            activities = activities.filter((activity) => activity.id !== id);
        } catch (error) {
            alert("Failed to delete activity.");
            console.error(error);
        }
    };

    // Start editing an activity
    const startEditing = (activity) => {
        editingActivity = { ...activity }; // Create a copy to avoid mutating the original directly
    };

    // Cancel editing
    const cancelEditing = () => {
        editingActivity = null;
    };

    // Navigate to the activity details page on clicking 'Meer info'
    const navigateToDetails = (id) => {
    goto(`/activities/${id}`);
};

    onMount(() => {
        fetchData();
    });
</script>

<div class="bg-green-500 min-h-screen text-white font-sans">
    <div class="px-4 py-6 text-center sm:text-left">
        <h1 class="text-2xl font-bold">Activiteiten</h1>
        <p class="mt-2 text-lg">In de buurt:</p>
    </div>

    <div class="bg-white rounded-t-3xl text-black px-4 py-6 space-y-4">
        {#each activities as activity}
            <div class="border rounded-lg shadow-sm p-4 flex flex-col sm:flex-row justify-between items-start sm:items-center">
                {#if editingActivity && editingActivity.id === activity.id}
                    <div class="w-full space-y-2">
                        <input
                            type="text"
                            bind:value={editingActivity.title}
                            placeholder="Titel"
                            class="w-full border rounded-lg p-2 text-sm text-gray-500"
                        />
                        <input
                            type="text"
                            bind:value={editingActivity.date}
                            placeholder="Datum of tijd"
                            class="w-full border rounded-lg p-2 text-sm text-gray-500"
                        />
                        <input
                            type="text"
                            bind:value={editingActivity.link}
                            placeholder="Link (optioneel)"
                            class="w-full border rounded-lg p-2 text-sm text-gray-500"
                        />
                        <div class="flex flex-wrap space-x-2">
                            <button
                                on:click={editActivity}
                                class="bg-green-500 text-black px-4 py-2 rounded-lg w-full sm:w-auto text-sm font-medium hover:bg-green-600"
                            >
                                Opslaan
                            </button>
                            <button
                                on:click={cancelEditing}
                                class="bg-gray-500 text-black px-4 py-2 rounded-lg w-full sm:w-auto text-sm font-medium hover:bg-gray-600"
                            >
                                Annuleren
                            </button>
                        </div>
                    </div>
                {:else}
                    <div class="flex-1">
                        <h2 class="text-lg font-semibold">{activity.title}</h2>
                        <p class="text-sm text-gray-600 whitespace-pre-line">{activity.date}</p>
                    </div>
                    <div class="flex flex-col sm:flex-row sm:space-x-2 space-y-2 sm:space-y-0">
                        <button
                            on:click={() => navigateToDetails(activity.id)}
                            class="bg-blue-500 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-blue-600 text-center"
                        >
                            Meer info
                        </button>
                        <button
                            on:click={() => startEditing(activity)}
                            class="bg-green-600 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-green-700"
                        >
                            Bewerken
                        </button>
                        <button
                            on:click={() => deleteActivity(activity.id)}
                            class="bg-red-500 text-white px-4 py-2 rounded-lg text-sm font-medium hover:bg-red-600"
                        >
                            Verwijderen
                        </button>
                    </div>
                {/if}
            </div>
        {/each}
    </div>

    <div class="bg-white px-4 py-6">
        <h2 class="text-lg font-bold text-green-500">Nieuwe activiteit toevoegen</h2>
        <div class="mt-4 space-y-2">
            <input
                type="text"
                bind:value={newTitle}
                placeholder="Titel"
                class="w-full border rounded-lg p-2 text-sm text-gray-500"
            />
            <input
                type="text"
                bind:value={newDate}
                placeholder="Datum of tijd"
                class="w-full border rounded-lg p-2 text-sm text-gray-500"
            />
            <input
                type="text"
                bind:value={newLink}
                placeholder="Link (optioneel)"
                class="w-full border rounded-lg p-2 text-sm text-gray-500"
            />
            <button
                on:click={addActivity}
                class="bg-green-500 text-black px-4 py-2 rounded-lg w-full text-sm font-medium hover:bg-green-600"
            >
                Toevoegen
            </button>
        </div>
    </div>
</div>
