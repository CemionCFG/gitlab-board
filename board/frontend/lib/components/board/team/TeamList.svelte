<script>
    import {Accordion, AccordionItem, Button, Input, InputGroup, Form} from "sveltestrap";

    import {projects, teams, teamprojects} from "../../../store";
    import {addTeam, attachTeamProject, deleteTeam, detachTeamProject} from "../../../api/board";


    let addTeamName = "";

    const handleDeleteTeam = async (id) => {
        const res = await deleteTeam(id);

        if (res.ok) {
            $teams = $teams.filter(t => t.id != id);
        }
    }

    const handleAddTeam = async (e) => {
        e.preventDefault();

        if (addTeamName.length > 1) {
            const res = await addTeam(addTeamName);
            const json_res = await res.json();

            if (res.ok) {
                $teams = [...$teams, json_res.data];
                $teamprojects[json_res.data.id] = [];
                addTeamName = "";
            }
        }
    }

    const handleCheckTeamProject = async (team_id, project_id, e) => {
        if (e.target.checked) {
            await attachTeamProject(team_id, project_id);

            $teamprojects[team_id].push(project_id);
            $teamprojects = $teamprojects;
        } else {
            await detachTeamProject(team_id, project_id);

            $teamprojects[team_id].splice($teamprojects[team_id].indexOf(project_id), 1);
            $teamprojects = $teamprojects;
        }
    }

    $: projectInTeam = (team_id, project_id) => $teamprojects[team_id].includes(project_id);
</script>

<div class="my-4">
    <Form on:submit={e => handleAddTeam(e)}>
        <InputGroup>
            <Input bind:value={addTeamName} placeholder="Team name"/>
            <Button color="primary" on:click={handleAddTeam}>Add</Button>
        </InputGroup>
    </Form>
</div>

<Accordion>
    {#each $teams as t (t.id)}
        <AccordionItem header={t.name}>
            <div class="d-flex justify-content-between">
                <div>
                    <h5>Projects</h5>
                    <p>Choose projects to show.</p>
                </div>
            </div>
            <div class="projects mb-2">
                {#each $projects as project}
                    <div class="form-check">
                            <input class="form-check-input" type="checkbox" id="{project.name}-{t.id}-Checkbox"
                                   on:change={(e) => handleCheckTeamProject(t.id, project.id, e)}
                                   checked="{projectInTeam(t.id, project.id)}"/>
                        <label class="form-check-label" for="{project.name}-{t.id}-Checkbox">{project.name_with_namespace}</label>
                    </div>
                {/each}
            </div>
            <div class="d-flex justify-content-between">
                <div>
                    <h5>Advanced</h5>
                </div>
            </div>
            <a href="#" class="text-danger" on:click={() => handleDeleteTeam(t.id)}>Delete Project</a>
        </AccordionItem>
    {/each}
</Accordion>


<style>
    .projects {
        height: 13.2rem;
        overflow: scroll;
    }
</style>
