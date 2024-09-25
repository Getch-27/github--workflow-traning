# Automated Testing Workflow

This repository contains an automated workflow that triggers tests when changes are made to specific parts of the codebase: **adapters**, **writers**, **config file**, and the **main script**. The workflow is designed to run tests efficiently based on the nature of the changes.

## Workflow Overview

The workflow performs the following steps:

1. **Trigger Conditions**:
   - The workflow runs on **pushes** or **pull requests** to the `main` branch.
   - It monitors changes in the `adapters/`, `writers/`, `config/`, and `main_script.py` files.

2. **Determine Changes**:
   - It identifies which files were modified between the current commit and the previous one.
   - Sets environment variables (`changed_adapters`, `changed_writers`, `changed_config`, `changed_main_script`) based on detected changes.

3. **Test Execution**:
   - **Both Writer & Config Changed**: prioritize the writer change
and run all adapters.
   - **Only Writer Changed**: Runs tests for all adapters since the writer change can impact how data is written across all components.
   - **Only Config Changed**: Runs tests for the specific parts of the system affected by the config changes.
   - **Only Main Script Changed**: Runs tests for all adapters with the default config.

## Updated code Workflow

```yaml
 # Prioritize writer when both writer and config change
          
          if [ -n "$CHANGED_WRITERS" ] && [ "$CONFIG_CHANGED" == "true" ]; then
            echo "Both writer and config changed. Prioritizing writer and running all adapters."
            poetry run python create_knowledge_graph.py --output-dir output --adapters-config config/adapters_config_sample.yaml --dbsnp-rsids aux_files/ensembl_to_hgnc.pkl --dbsnp-pos aux_files/hgnc_to_ensembl.pkl --writer-type $CURRENT_WRITER
          elif [ "$MAIN_SCRIPT_CHANGED" == "true" ]; then
            echo "Running with full sample config due to main script change"
            poetry run python create_knowledge_graph.py --output-dir output --adapters-config config/adapters_config_sample.yaml --dbsnp-rsids aux_files/ensembl_to_hgnc.pkl --dbsnp-pos aux_files/hgnc_to_ensembl.pkl --writer-type $CURRENT_WRITER
          elif [ -n "$CHANGED_WRITERS" ]; then
            if [[ $CHANGED_WRITERS == *"$CURRENT_WRITER"* ]] || [ "$CHANGED_WRITERS" == "$ALL_WRITERS" ]; then
              echo "Running all adapters with changed writer: $CURRENT_WRITER"
              poetry run python create_knowledge_graph.py --output-dir output --adapters-config config/test_config.yaml --dbsnp-rsids aux_files/ensembl_to_hgnc.pkl --dbsnp-pos aux_files/hgnc_to_ensembl.pkl --writer-type $CURRENT_WRITER
            else
              echo "Skipping unchanged writer: $CURRENT_WRITER"
            fi
           elif [ -n "$CHANGED_ADAPTERS" ] || [ "$CONFIG_CHANGED" == "true" ]; then
            echo "Running changed adapters or config with writer: $CURRENT_WRITER"
            poetry run python create_knowledge_graph.py --output-dir output --adapters-config config/test_config.yaml --dbsnp-rsids aux_files/ensembl_to_hgnc.pkl --dbsnp-pos aux_files/hgnc_to_ensembl.pkl --writer-type $CURRENT_WRITER
          else
             echo "No relevant changes detected for this writer. Skipping."
          fi
    

