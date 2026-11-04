# DeepSkyMotion_scripts

Scripts for PixInsight, distributed as a PixInsight update repository.

## Install

1. In PixInsight open **Resources → Updates → Manage Repositories**.
2. Click **Add** and enter:

   ```
   https://raw.githubusercontent.com/DeepSkyMotion/DeepSkyMotion_scripts/main/
   ```

3. Run **Resources → Updates → Check for Updates** and restart PixInsight.

Scripts appear under **Script → DeepSkyMotion**.

## Layout

- `updates.xri` — repository index read by PixInsight.
- `*.zip` — script packages; their contents mirror the PixInsight install folder
  (`src/scripts/DeepSkyMotion/<Script>/...`).
