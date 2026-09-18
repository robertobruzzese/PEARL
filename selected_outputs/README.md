# PEARL selected CSV outputs

Curated CSVs from the PEARL pipelines, selected for sharing and for creating peptide and molecule figures.

Source: local PEARL `outputs/` directory. Paths in the table are relative to that directory.

A file is copied only when a non-test version is available. If several non-test copies exist, a production copy is preferred; remaining ties are resolved alphabetically by source path. Original CSV names are preserved.

These CSVs may refer to structural files or trajectories that are not included here.

## Copied files

| Pipeline | CSV | Source relative to outputs |
|---|---|---|
| pipeline_1 | `candidate_seed_peptides.csv` | `candidate_seed_peptides.csv` |
| pipeline_1 | `diffusion_peptide_candidates.csv` | `diffusion_peptide_candidates.csv` |
| pipeline_1 | `diffusion_energetic_peptide_candidates.csv` | `diffusion_energetic_peptide_candidates.csv` |
| pipeline_1 | `contiguous_windows_ranked.csv` | `contiguous_windows_ranked.csv` |
| pipeline_1 | `energetic_hotspots.csv` | `energetic_hotspots.csv` |
| pipeline_1 | `interface_residues_with_foldx_ddg.csv` | `interface_residues_with_foldx_ddg.csv` |
| pipeline_2 | `clear_local_peptide_variant_dataset_labeled.csv` | `clear_local_peptide_dataset_04c/clear_local_peptide_variant_dataset_labeled.csv` |
| pipeline_2 | `oracle_all_predictions.csv` | `clear_peptide_oracle_04d/oracle_all_predictions.csv` |
| pipeline_2 | `clear_all_counterfactual_candidates.csv` | `clear_peptide_counterfactuals_04e/clear_all_counterfactual_candidates.csv` |
| pipeline_2 | `clear_counterfactual_final_ranking.csv` | `clear_peptide_counterfactuals_04e/clear_counterfactual_final_ranking.csv` |
| pipeline_2 | `clear_counterfactual_foldx_structural_validation.csv` | `clear_counterfactual_validation_05c/clear_counterfactual_foldx_structural_validation.csv` |
| pipeline_2 | `clear_counterfactual_flexpepdock_final_ranking.csv` | `clear_counterfactual_flexpepdock_05d/clear_counterfactual_flexpepdock_final_ranking.csv` |
| pipeline_3 | `07d_selected_peptide_MD_comparison.csv` | `pipeline_3_md_07d_selected_peptide_comparison/production_1ns/07d_selected_peptide_MD_comparison.csv` |
| pipeline_3 | `candidate_specific_persistent_receptor_contacts.csv` | `pipeline_3_md_07d_selected_peptide_comparison/production_1ns/candidate_specific_persistent_receptor_contacts.csv` |
| pipeline_3 | `07e_endpoint_energy_summary.csv` | `pipeline_3_md_07e_mmgbsa_endpoint_comparison/tables/07e_endpoint_energy_summary.csv` |
| pipeline_3 | `07e_dynamic_energetic_integration.csv` | `pipeline_3_md_07e_mmgbsa_endpoint_comparison/tables/07e_dynamic_energetic_integration.csv` |
| pipeline_3 | `07f_FINAL_integrated_candidate_comparison.csv` | `pipeline_3_md_07f_cf05_followup/production_1ns/tables/07f_FINAL_integrated_candidate_comparison.csv` |
| pipeline_4 | `08a_ESM2_ranking.csv` | `pipeline_4_ai_08a_esm2_sequence_scoring/tables/08a_ESM2_ranking.csv` |
| pipeline_4 | `08b_ProteinMPNN_unique_designs.csv` | `pipeline_4_ai_08b_proteinmpnn/production/tables/08b_ProteinMPNN_unique_designs.csv` |
| pipeline_4 | `08b_ProteinMPNN_shortlist_for_08c.csv` | `pipeline_4_ai_08b_proteinmpnn/production/tables/08b_ProteinMPNN_shortlist_for_08c.csv` |
| pipeline_4 | `08c_AI_integrated_ranking.csv` | `pipeline_4_ai_08c_integrated_candidate_selection/tables/08c_AI_integrated_ranking.csv` |
| pipeline_4 | `08c_final_new_candidate_shortlist.csv` | `pipeline_4_ai_08c_integrated_candidate_selection/tables/08c_final_new_candidate_shortlist.csv` |
| pipeline_4 | `08d_integrated_structural_ranking.csv` | `pipeline_4_ai_08d_structural_validation/tables/08d_integrated_structural_ranking.csv` |
| pipeline_4 | `08j_final_multiobjective_ranking.csv` | `pipeline_4_ai_08j_final_multiobjective/tables/08j_final_multiobjective_ranking.csv` |
| pipeline_5 | `09a_core_consensus_pharmacophore.csv` | `pipeline_5_pharmacophore_09a_md_derived/production/tables/09a_core_consensus_pharmacophore.csv` |
| pipeline_5 | `09b_screening_ready_pharmacophore.csv` | `pipeline_5_pharmacophore_09b_consolidated/tables/09b_screening_ready_pharmacophore.csv` |
| pipeline_5 | `09c_generated_or_acquired_smiles.csv` | `pipeline_5_pharmacophore_09c_generation/production/tables/09c_generated_or_acquired_smiles.csv` |
| pipeline_5 | `09c_candidate_ranking.csv` | `pipeline_5_pharmacophore_09c_generation/production/tables/09c_candidate_ranking.csv` |
| pipeline_5 | `09d_docking_shortlist.csv` | `pipeline_5_pharmacophore_09d_chemical_filtering/tables/09d_docking_shortlist.csv` |
| pipeline_5 | `09e_best_pose_per_candidate.csv` | `pipeline_5_pharmacophore_09e_diffdock/tables/09e_best_pose_per_candidate.csv` |
| pipeline_5 | `09f_vina_refined_ranking.csv` | `pipeline_5_pharmacophore_09f_vina_refinement/tables/09f_vina_refined_ranking.csv` |
| pipeline_5 | `09g_integrated_lead_priority.csv` | `pipeline_5_pharmacophore_09g_integrated_lead_selection/tables/09g_integrated_lead_priority.csv` |
| pipeline_5 | `09g_structure_index.csv` | `pipeline_5_pharmacophore_09g_integrated_lead_selection/tables/09g_structure_index.csv` |

## Missing files

None.

Copied: 33 CSVs. Missing: 0 CSVs.
