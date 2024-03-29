<script lang="ts">
	import * as Card from "$lib/components/ui/card";
	import * as AlertDialog from "$lib/components/ui/alert-dialog";
	import { Button } from "$lib/components/ui/button";
	import type { PostWithUser } from "$lib/server/schemas";
	import * as DropdownMenu from "$lib/components/ui/dropdown-menu";
	import { EllipsisVertical, Trash2, SquarePen, Square } from "lucide-svelte";
	import { buttonVariants } from "./ui/button";
	import { type SuperValidated, type Infer, superForm } from "sveltekit-superforms";
	import { deletePostSchema } from "$lib/zod-schema";
	import { zodClient } from "sveltekit-superforms/adapters";
	import { sleep } from "$lib/utils.js";
	import { toast } from "svelte-sonner";
	import { page } from "$app/stores";

	type Props = {
		post: PostWithUser;
		form: SuperValidated<Infer<typeof deletePostSchema>>;
	};
	let { post, form: theForm } = $props<Props>();

	const form = superForm(theForm, {
		validators: zodClient(deletePostSchema),
		onUpdated: ({ form: returnForm }) => {
			if (!returnForm.valid) return toast.error("Error deleting your post!");
			openStates.deleteDialogOpen = false;
			toast.success("Post deleted successfully!");
		},
	});

	const { enhance } = form;

	const openStates = $state({
		deleteDialogOpen: false,
		dropdownOpen: false,
		editDialogOpen: false,
	});

	$page;
</script>

<Card.Root>
	<Card.Header class="flex-row items-center justify-between">
		<Card.Title>
			{post.title}
		</Card.Title>
		{#if $page.data.user && $page.data.user.id === post.userId}
			<DropdownMenu.Root bind:open={openStates.dropdownOpen}>
				<DropdownMenu.Trigger class={buttonVariants({ size: "icon", variant: "ghost" })}>
					<EllipsisVertical class="size-4" />
					<span class="sr-only">Post options</span>
				</DropdownMenu.Trigger>
				<DropdownMenu.Content>
					<DropdownMenu.Item href="/posts/{post.id}/edit">
						<SquarePen class="mr-2 size-4" />
						Edit
					</DropdownMenu.Item>
					<DropdownMenu.Item
						onclick={(e) => {
							e.preventDefault();
							openStates.dropdownOpen = false;
							sleep(2).then(() => {
								openStates.deleteDialogOpen = true;
							});
						}}
					>
						<Trash2 class="mr-2 size-4" />
						Delete
					</DropdownMenu.Item>
				</DropdownMenu.Content>
			</DropdownMenu.Root>
		{/if}
	</Card.Header>
	<Card.Content>
		{post.content}
	</Card.Content>
	<Card.Footer>
		{post.user.username}
	</Card.Footer>
</Card.Root>

<AlertDialog.Root bind:open={openStates.deleteDialogOpen}>
	<AlertDialog.Content>
		<AlertDialog.Header>
			<AlertDialog.Title>Delete Post</AlertDialog.Title>
			<AlertDialog.Description>Are you sure you want to delete this post?</AlertDialog.Description>
		</AlertDialog.Header>
		<AlertDialog.Footer>
			<form use:enhance method="POST" action="?/deletePost&id={post.id}">
				<Button class={buttonVariants({ variant: "destructive" })} type="submit">
					Yes, delete.
				</Button>
			</form>
			<AlertDialog.Cancel
				class={buttonVariants({ variant: "outline" })}
				onclick={() => (openStates.deleteDialogOpen = false)}>No, cancel.</AlertDialog.Cancel
			>
		</AlertDialog.Footer>
	</AlertDialog.Content>
</AlertDialog.Root>
