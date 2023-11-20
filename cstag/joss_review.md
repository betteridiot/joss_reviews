# [betteridiot](https://github.com/betteridiot) JoSS review of [`cstag`](https://github.com/akikuno/cstag) and [`cstag-cli`](https://github.com/akikuno/cstag-cli)

Submitting author: @akikuno (Akihiro Kuno )<br/>
Repository: https://github.com/akikuno/cstag<br/>
Version: v3.2.2<br/>
Editor: @jbloom<br/>
Reviewer: @betteridiot<br/>

---
# Quick Checks:
1. General Stuff:
    - [x] Software was available at the listed repo
    - [x] MIT license present
    - [x] Version (v1.0.4) **did not** match the repo (v1.0.5)
    - [x] Authorship ([@akikuno](https://github.com/akikuno)) was appropriate
2. Documentation:
    - [ ] Statement of need was present and appropriate
    * Documentation does not express statement of need. It only describes what the program does.
    - [ ] Installation instructions clearly stated?
    * Please see notes under GitHub and PIP [installation checks](##github-install)
    - [x] Example usage was present
    - [x] Satisfactory functionality documentation
    - [ ] Automated tests
    * Doctests for `cstag` do not pass do to minor syntax errors. See [Testing doctests](##testing-doctests)
    - [x] Community guidelines
3. Software paper:
    - [x] Authors and affiliations present and appropriate
    - [x] Statement of need present within the paper
    - [ ] State of the field
    * Acknowledges `2passtools` but does not address its internal [cs tag functionality](https://github.com/bartongroup/2passtools/blob/d4378d0f4a780b5b89399161ed8eac6ca5092128/lib2pass/bamparse.py#L13-L56)
    * I am curious about the author's "rebranding" of [`calcs`](https://pypi.org/project/calcs/), which appears to be a precursor to `cstag-cli`. Maybe add a deconfliction/redirect statement to one of the packages involved.
    * As addressed by @jbloom, the author should recognize [`alignparse`](https://github.com/jbloomlab/alignparse/blob/b0d72945421b361cfed682e2c4d7dfb38ec1842c/alignparse/cs_tag.py) in their state of the field. 
    - [x] Quality of writing
    - [ ] References
          * Missing DOIs for `minimap2` and Bioconda.
        
#### **Notes** on Quick Check items:
This is a minor thing that could lead people astray. The `CS` (uppercase) is a reserved for color read sequence information as per the [SAM format](https://samtools.github.io/hts-specs/SAMtags.pdf), while the `cs` (lowercase)
tag represents the difference string. This has already been a [topic of discussion](https://github.com/samtools/samtools/issues/1030#issuecomment-477137623) while feature requests to incorporate the `cs` tag more 
formallying into the SAM format.

---
# Installation:
Below are the steps I used to setup my environment for this review:

## Bioconda build
```bash
# Mark my current environment
$ conda info
    ...
    conda version: 23.10.0
    ...
    python version: 3.11.6.final.0
    ...
    platform: linux-64
 
$ conda create -n joss_cstag --no-default-packages --solver libmamba bioconda::cstag bioconda::cstag-cli
$ conda activate joss_cstag
```
:heavy_check_mark: Bioconda installation worked as expected.

## GitHub install
```bash
# Clean enviornment
$ conda remove -n joss_cstag --all
$ conda create -n joss_cstag --no-default-packages python
$ conda activate joss_cstag

# Get cstag and cstag-cli
$ git clone https://github.com/akikuno/cstag.git
$ git clone https://github.com/akikuno/cstag-cli.gi
$ cd cstag
$ python -m pip install . # passes

$ cd ../cstag-cli
$ python -m pip install . # Fails
...
checking for zlib.h... no
checking for inflate in -lz... no
configure: error: zlib development files not found

HTSlib uses compression routines from the zlib library <http://zlib.net>.
Building HTSlib requires zlib development files to be installed on the build
machine; you may need to ensure a package such as zlib1g-dev (on Debian or
Ubuntu Linux) or zlib-devel (on RPM-based Linux distributions or Cygwin)
is installed.

FAILED.  This error must be resolved in order to build HTSlib successfully.
...
error: subprocess-exited-with-error

× Getting requirements to build wheel did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.
```
* :heavy_check_mark: Installation of `cstag` from `cstag/pyproject.toml` worked as intended.
* :triangular_flag_on_post: Installation of `cstag-cli` from `cstag-cli/pyproject.toml` **did not** work as intended.
    * The issue comes from the requirement of `pysam` which requires `htslib` which requires `zlib.h`. These dependencies are managed well by `conda` through Bioconda, but cannot be installed as an editable
    repository. While not documented as an intended installation option, I believe that it is important to note. This dependency issue was one of the principal reasons that I wrote `bamnostic`. It isn't a
    wrapper for `htslib`, so it has no requirements for it. This also allows it to be installed on any system that can have Python. This isn't a sales pitch though and, by no means, an indictment on `htslib`
    or `pysam`. These are terrific packages that serve the community/field in an essential way. But, as of [`htslib` v1.18](https://anaconda.org/bioconda/htslib) on Bioconda, Windows support is still missing.
    * More importantly, when regarding this review, `cstag-cli`'s dependencies to `pysam` (and `htslib` by assocation) prevent editable installs or installs on systems whereas the user does not have sudo privileges or access
    to `conda`/`mamba` since they cannot install `zlib.h` (easily) without them. Just something to consider.

## `pip` build
```bash
$ conda remove -n joss_cstag --all
$ conda create -n joss_cstag --solver libmamba --no-default-packages python
$ conda activate joss_cstag
$ pip install cstag
$ pip install cstag-cli
...
checking for zlib.h... no
checking for inflate in -lz... no
configure: error: zlib development files not found

HTSlib uses compression routines from the zlib library <http://zlib.net>.
Building HTSlib requires zlib development files to be installed on the build
machine; you may need to ensure a package such as zlib1g-dev (on Debian or
Ubuntu Linux) or zlib-devel (on RPM-based Linux distributions or Cygwin)
is installed.

FAILED.  This error must be resolved in order to build HTSlib successfully.
...
error: subprocess-exited-with-error

× Getting requirements to build wheel did not run successfully.
│ exit code: 1
╰─> See above for output.

note: This error originates from a subprocess, and is likely not a problem with pip.

$ conda install --solver libmamba conda-forge::zlib && pip install cstag-cli
```
:triangular_flag_on_post: It appears the direct `pip` install from PyPI has the same issue as above regarding installation dependencies (not surprising). By installing `zlib` through `conda-forge` on a system that does not have `zlib.h` available,
one can overcome this issue and successfully install `cstag-cli`.

This dependency issue does not rest with your program in any way. It is a product of `pysam`/`htslib` dependencies. I also acknowledge that most people in bioinformatics don't often has as sterile of an environment as I do. 
I just suspect that there might be some graduate student out there that has only lightly used `conda`, `pip`, or any other package manager could potentially be stumped by this. 

My suggestion is to make the `pysam` (and, by extension, `htslib`) dependencies more explicitly documented on your repo/`README.md` since these dependencies cannot be address by `pip` alone. However, they are properly managed
by `conda`/`mamba`. While an edge case, it is still a case.

From this point on, I will be using the [GitHub build](##github-install) for Bioconda build for review and the GitHub build for testing.

---
# Testing 
## Testing the general import

```bash
$ python -c "import cstag; print(True)"
True
$ cstag --help
```
:heavy_check_mark: Basic import of `cstag` ran without error.
:heavy_check_mark: Help prompt of `cstag-cli` ran without error.

---
## Testing with `pytest`
:triangular_flag_on_post: Missing documentation on testing the build in `README.md`.
:triangular_flag_on_post: Tests not included in package installs. Consider adding `tests` to your [`pyproject.toml`](https://python-poetry.org/docs/pyproject/#include-and-exclude) under the `include` parameter.
 
```bash
$ conda install --solver libmamba pytset
$ pytest --version
pytest 7.4.3
$ cd Downloads/cstag
$ pytest -vv tests/
================================================= test session starts ==================================================
platform linux -- Python 3.12.0, pytest-7.4.3, pluggy-1.3.0 -- /<PATH>/miniconda3/envs/joss_cstag_test/bin/python3.12
cachedir: .pytest_cache
rootdir: /<PATH>/Downloads/cstag/cstag
collected 149 items

tests/test_call.py::testparse_cigar[5S10M3S-expected0] PASSED                                                    [  0%]
tests/test_call.py::testparse_cigar[10M-expected1] PASSED                                                        [  1%]
tests/test_call.py::testparse_cigar[-expected2] PASSED                                                           [  2%]
tests/test_call.py::testparse_cigar[5H10M5H-expected3] PASSED                                                    [  2%]
tests/test_call.py::testparse_md[6^ATA14A3-expected_output0] PASSED                                              [  3%]
tests/test_call.py::testparse_md[-expected_output1] PASSED                                                       [  4%]
tests/test_call.py::testparse_md[10-expected_output2] PASSED                                                     [  4%]
tests/test_call.py::testparse_md[^ACGT-expected_output3] PASSED                                                  [  5%]
tests/test_call.py::testparse_md[AGCT-expected_output4] PASSED                                                   [  6%]
tests/test_call.py::testparse_md[5^AC5T2-expected_output5] PASSED                                                [  6%]
tests/test_call.py::testparse_md[1A1^T1-expected_output6] PASSED                                                 [  7%]
tests/test_call.py::testparse_md[100-expected_output7] PASSED                                                    [  8%]
tests/test_call.py::testparse_md[^ACGTACGT-expected_output8] PASSED                                              [  8%]
tests/test_call.py::testparse_md[1ATGC1-expected_output9] PASSED                                                 [  9%]
tests/test_call.py::testjoin_cigar[cigar_tuples0-5H10M5S] PASSED                                                 [ 10%]
tests/test_call.py::testjoin_cigar[cigar_tuples1-10M] PASSED                                                     [ 10%]
tests/test_call.py::testjoin_cigar[cigar_tuples2-] PASSED                                                        [ 11%]
tests/test_call.py::testjoin_cigar[cigar_tuples3-5H10M5H] PASSED                                                 [ 12%]
tests/test_call.py::testjoin_cigar[cigar_tuples4-5S10M] PASSED                                                   [ 12%]
tests/test_call.py::testjoin_cigar[cigar_tuples5-10M5S] PASSED                                                   [ 13%]
tests/test_call.py::testjoin_cigar[cigar_tuples6-5H10M5S] PASSED                                                 [ 14%]
tests/test_call.py::testjoin_cigar[cigar_tuples7-5S10M5H] PASSED                                                 [ 14%]
tests/test_call.py::test_trim_clips[5S10M3S-AAAAATTTTTGGGGGCCC-10M-TTTTTGGGGG] PASSED                            [ 15%]
tests/test_call.py::test_trim_clips[10M-TTTTTTTTTT-10M-TTTTTTTTTT] PASSED                                        [ 16%]
tests/test_call.py::test_trim_clips[5H10M5H-TTTTTTTTTT-10M-TTTTTTTTTT] PASSED                                    [ 16%]
tests/test_call.py::test_trim_clips[5S10M-AAAAATTTTTTTTTT-10M-TTTTTTTTTT] PASSED                                 [ 17%]
tests/test_call.py::test_trim_clips[10M3S-TTTTTTTTTTGGG-10M-TTTTTTTTTT] PASSED                                   [ 18%]
tests/test_call.py::test_trim_clips[5S10M5H-AAAAATTTTTTTTTT-10M-TTTTTTTTTT] PASSED                               [ 18%]
tests/test_call.py::test_trim_clips[5H10M5S-TTTTTTTTTTGGGGG-10M-TTTTTTTTTT] PASSED                               [ 19%]
tests/test_call.py::test_generate_cs_tag_long_form[8M2D4M2I-8^AG6-ACGTACGTACGTAC-=ACGTACGT-ag=ACGT+ac] PASSED    [ 20%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-5-ACGTA-=ACGTA] PASSED                                     [ 20%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-3C1-ACGTG-=ACG*ct=G] PASSED                                [ 21%]
tests/test_call.py::test_generate_cs_tag_long_form[5M1I3M-9-ACGTAGCTA-=ACGTA+g=CTA] PASSED                       [ 22%]
tests/test_call.py::test_generate_cs_tag_long_form[5M1D4M-5^A4-ACGTGGCTA-=ACGTG-a=GCTA] PASSED                   [ 22%]
tests/test_call.py::test_generate_cs_tag_long_form[3M1D1M1D4M-3^C1^A4-ACGGCTAG-=ACG-c=G-a=CTAG] PASSED           [ 23%]
tests/test_call.py::test_generate_cs_tag_long_form[3S5M-5-NNNACGTA-=ACGTA] PASSED                                [ 24%]
tests/test_call.py::test_generate_cs_tag_long_form[8M2D4M2I3N1M-2A5^AG7-ACGTACGTACGTACG-=AC*ag=TACGT-ag=ACGT+ac~nn3nn=G] PASSED [ 24%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-0C4-ACGTA-*ca=CGTA] PASSED                                 [ 25%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-4C0-ACGTA-=ACGT*ca] PASSED                                 [ 26%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-0C3C0-ACGTA-*ca=CGT*ca0] PASSED                            [ 26%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-0C3C0-ACGTA-*ca=CGT*ca1] PASSED                            [ 27%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-2CC1-ACGTA-=AC*cg*ct=A] PASSED                             [ 28%]
tests/test_call.py::test_generate_cs_tag_long_form[3M2D1M2I1M3N1M-3^GG0A2-AAATCCTT-=AAA-gg*at+cc=T~nn3nn=T] PASSED [ 28%]
tests/test_call.py::test_generate_cs_tag_short_form[8M2D4M2I-8^AG6-ACGTACGTACGTAC-:8-ag:4+ac] PASSED             [ 29%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-5-ACGTA-:5] PASSED                                        [ 30%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-3C1-ACGTG-:3*ct:1] PASSED                                 [ 30%]
tests/test_call.py::test_generate_cs_tag_short_form[5M1I3M-9-ACGTAGCTA-:5+g:3] PASSED                            [ 31%]
tests/test_call.py::test_generate_cs_tag_short_form[5M1D4M-5^A4-ACGTGGCTA-:5-a:4] PASSED                         [ 32%]
tests/test_call.py::test_generate_cs_tag_short_form[3M1D1M1D4M-3^C1^A4-ACGGCTAG-:3-c:1-a:4] PASSED               [ 32%]
tests/test_call.py::test_generate_cs_tag_short_form[3S5M-5-NNNACGTA-:5] PASSED                                   [ 33%]
tests/test_call.py::test_generate_cs_tag_short_form[8M2D4M2I3N1M-2A5^AG7-ACGTACGTACGTACG-:2*ag:5-ag:4+ac~nn3nn:1] PASSED [ 34%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-0C3C0-ACGTA-*ca:3*ca] PASSED                              [ 34%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-2CC1-ACGTA-:2*cg*ct:1] PASSED                             [ 35%]
tests/test_call.py::test_generate_cs_tag_short_form[3M2D1M2I1M3N1M-3^GG0A2-AAATCCTT-:3-gg*at+cc:1~nn3nn:1] PASSED [ 36%]
tests/test_consensus.py::test_split_cs_tags PASSED                                                               [ 36%]
tests/test_consensus.py::test_normalize_read_lengths PASSED                                                      [ 37%]
tests/test_consensus.py::test_get_consensus PASSED                                                               [ 38%]
tests/test_consensus.py::test_substitution PASSED                                                                [ 38%]
tests/test_consensus.py::test_insertion PASSED                                                                   [ 39%]
tests/test_consensus.py::test_deletion PASSED                                                                    [ 40%]
tests/test_consensus.py::test_splicing PASSED                                                                    [ 40%]
tests/test_consensus.py::test_positions PASSED                                                                   [ 41%]
tests/test_consensus.py::test_positions_more_than_one PASSED                                                     [ 42%]
tests/test_lengthen.py::test_mutation PASSED                                                                     [ 42%]
tests/test_lengthen.py::test_softclip PASSED                                                                     [ 43%]
tests/test_lengthen.py::test_substitution PASSED                                                                 [ 44%]
tests/test_lengthen.py::test_deletion PASSED                                                                     [ 44%]
tests/test_lengthen.py::test_insertion PASSED                                                                    [ 45%]
tests/test_lengthen.py::test_splicing PASSED                                                                     [ 46%]
tests/test_lengthen.py::test_5bpIns_3bp_Del PASSED                                                               [ 46%]
tests/test_lengthen.py::test_first_5nt_del_softclip_5nt PASSED                                                   [ 47%]
tests/test_lengthen.py::test_softclip_plus_5nt PASSED                                                            [ 48%]
tests/test_lengthen.py::test_softclip_plusminus_10nt PASSED                                                      [ 48%]
tests/test_lengthen.py::test_real PASSED                                                                         [ 49%]
tests/test_mask.py::test_basic PASSED                                                                            [ 50%]
tests/test_mask.py::test_basic_with_prefix PASSED                                                                [ 51%]
tests/test_mask.py::test_all_n PASSED                                                                            [ 51%]
tests/test_mask.py::test_softclip PASSED                                                                         [ 52%]
tests/test_mask.py::test_splicing PASSED                                                                         [ 53%]
tests/test_mask.py::test_threshold_2 PASSED                                                                      [ 53%]
tests/test_mask.py::test_threshold_15 PASSED                                                                     [ 54%]
tests/test_mask.py::test_error_cstag_short PASSED                                                                [ 55%]
tests/test_mask.py::test_error_threshold_float PASSED                                                            [ 55%]
tests/test_mask.py::test_error_threshold_45 PASSED                                                               [ 56%]
tests/test_revcomp.py::test_revcomp_normal PASSED                                                                [ 57%]
tests/test_revcomp.py::test_revcomp_prefix PASSED                                                                [ 57%]
tests/test_revcomp.py::test_revcomp_empty PASSED                                                                 [ 58%]
tests/test_revcomp.py::test_revcomp_special_chars PASSED                                                         [ 59%]
tests/test_revcomp.py::test_revcomp_invalid PASSED                                                               [ 59%]
tests/test_shorten.py::test_mutation PASSED                                                                      [ 60%]
tests/test_shorten.py::test_softclip PASSED                                                                      [ 61%]
tests/test_shorten.py::test_substitution PASSED                                                                  [ 61%]
tests/test_shorten.py::test_deletion PASSED                                                                      [ 62%]
tests/test_shorten.py::test_insertion PASSED                                                                     [ 63%]
tests/test_shorten.py::test_splicing PASSED                                                                      [ 63%]
tests/test_shorten.py::test_5bpIns_3bp_Del PASSED                                                                [ 64%]
tests/test_shorten.py::test_first_5nt_del_softclip_5nt PASSED                                                    [ 65%]
tests/test_shorten.py::test_softclip_plus_5nt PASSED                                                             [ 65%]
tests/test_shorten.py::test_softclip_plusminus_10nt PASSED                                                       [ 66%]
tests/test_shorten.py::test_real PASSED                                                                          [ 67%]
tests/test_split.py::test_split[-False-expected_output0] PASSED                                                  [ 67%]
tests/test_split.py::test_split[cs:Z:-False-expected_output1] PASSED                                             [ 68%]
tests/test_split.py::test_split[cs:Z:-True-expected_output2] PASSED                                              [ 69%]
tests/test_split.py::test_split[:4-False-expected_output3] PASSED                                                [ 69%]
tests/test_split.py::test_split[:4:3-False-expected_output4] PASSED                                              [ 70%]
tests/test_split.py::test_split[:4*ag:3-False-expected_output5] PASSED                                           [ 71%]
tests/test_split.py::test_split[:4+ag:3-False-expected_output6] PASSED                                           [ 71%]
tests/test_split.py::test_split[:4-ag:3-False-expected_output7] PASSED                                           [ 72%]
tests/test_split.py::test_split[:4~gt1ac:3-False-expected_output8] PASSED                                        [ 73%]
tests/test_split.py::test_split[:2*ag+t-ccc~gt1ac:2-False-expected_output9] PASSED                               [ 73%]
tests/test_split.py::test_split[=ACGT-False-expected_output10] PASSED                                            [ 74%]
tests/test_split.py::test_split[=ACGT=CGTA-False-expected_output11] PASSED                                       [ 75%]
tests/test_split.py::test_split[=ACGT*ag=CGT-False-expected_output12] PASSED                                     [ 75%]
tests/test_split.py::test_split[=ACGT+ag=CGT-False-expected_output13] PASSED                                     [ 76%]
tests/test_split.py::test_split[=ACGT-ag=CGT-False-expected_output14] PASSED                                     [ 77%]
tests/test_split.py::test_split[=ACGT~gt1ac=CGT-False-expected_output15] PASSED                                  [ 77%]
tests/test_split.py::test_split[=ACGT*ac+gg-cc=T-False-expected_output16] PASSED                                 [ 78%]
tests/test_split.py::test_split[=AC*ag+t-ccc~gt1ac=AC-False-expected_output17] PASSED                            [ 79%]
tests/test_to_html.py::test_append_mark_to_n PASSED                                                              [ 79%]
tests/test_to_html.py::test_split_cstag PASSED                                                                   [ 80%]
tests/test_to_html.py::test_html PASSED                                                                          [ 81%]
tests/test_to_html.py::test_html_repeat_substitution PASSED                                                      [ 81%]
tests/test_to_html.py::test_html_repeat_substitution_start PASSED                                                [ 82%]
tests/test_to_html.py::test_html_repeat_substitution_end PASSED                                                  [ 83%]
tests/test_to_html.py::test_html_start_from_N PASSED                                                             [ 83%]
tests/test_to_html.py::test_html_deletion_with_N PASSED                                                          [ 84%]
tests/test_to_html.py::test_html_N_within_deletions PASSED                                                       [ 85%]
tests/test_to_html.py::test_html_N_within_insertions PASSED                                                      [ 85%]
tests/test_to_sequence.py::test_to_sequence_normal_cases PASSED                                                  [ 86%]
tests/test_to_sequence.py::test_to_sequence_edge_cases PASSED                                                    [ 87%]
tests/test_to_vcf.py::test_find_ref_for_insertion PASSED                                                         [ 87%]
tests/test_to_vcf.py::test_find_ref_for_deletion PASSED                                                          [ 88%]
tests/test_to_vcf.py::test_get_variant_annotations PASSED                                                        [ 89%]
tests/test_to_vcf.py::test_get_pos_end PASSED                                                                    [ 89%]
tests/test_to_vcf.py::test_format_cs_tags PASSED                                                                 [ 90%]
tests/test_to_vcf.py::test_group_by_chrom PASSED                                                                 [ 91%]
tests/test_to_vcf.py::test_group_by_overlapping_intervals PASSED                                                 [ 91%]
tests/test_to_vcf.py::test_call_reference_depth PASSED                                                           [ 92%]
tests/test_to_vcf.py::test_add_vcf_fields PASSED                                                                 [ 93%]
tests/test_to_vcf.py::test_process_cs_tag PASSED                                                                 [ 93%]
tests/test_to_vcf.py::test_process_cs_tags_simple_case PASSED                                                    [ 94%]
tests/test_to_vcf.py::test_process_cs_tags_with_splice PASSED                                                    [ 95%]
tests/test_validator.py::test_validate_cs_tag_normal_cases PASSED                                                [ 95%]
tests/test_validator.py::test_validate_cs_tag_abnormal_cases PASSED                                              [ 96%]
tests/test_validator.py::test_validate_cs_tag_edge_cases PASSED                                                  [ 97%]
tests/test_validator.py::test_validate_short_format PASSED                                                       [ 97%]
tests/test_validator.py::test_validate_long_format PASSED                                                        [ 98%]
tests/test_validator.py::test_validate_threshold PASSED                                                          [ 99%]
tests/test_validator.py::test_validate_pos PASSED                                                                [100%]

================================================= 149 passed in 1.10s ==================================================
$ cd ../cstag-cli
$ pytest -vv tests/
================================================= test session starts ==================================================
platform linux -- Python 3.12.0, pytest-7.4.3, pluggy-1.3.0 -- /<PATH>/miniconda3/envs/joss_cstag_test/bin/python3.12
cachedir: .pytest_cache
rootdir: /<PATH>/Downloads/cstag/cstag-cli
collected 2 items

tests/utils/test_io.py::test_determine_format PASSED                                                             [ 50%]
tests/utils/test_io.py::test_read_sam_with_sam_file PASSED                                                       [100%]

================================================== 2 passed in 0.05s ===================================================
```
* :heavy_check_mark:`cstag` test suite passed.
* :heavy_check_mark:`cstag-cli` test suite passed.

---
## Testing code coverage

```bash
$ pip install pytest-cov
$ cd ../cstag
$ pytest -vv --cov
================================================= test session starts ==================================================
platform linux -- Python 3.12.0, pytest-7.4.3, pluggy-1.3.0 -- /<PATH>/miniconda3/envs/joss_cstag_test/bin/python3.12
cachedir: .pytest_cache
rootdir: /<PATH>/Downloads/cstag/cstag
plugins: cov-4.1.0
collected 149 items

tests/test_call.py::testparse_cigar[5S10M3S-expected0] PASSED                                                    [  0%]
tests/test_call.py::testparse_cigar[10M-expected1] PASSED                                                        [  1%]
tests/test_call.py::testparse_cigar[-expected2] PASSED                                                           [  2%]
tests/test_call.py::testparse_cigar[5H10M5H-expected3] PASSED                                                    [  2%]
tests/test_call.py::testparse_md[6^ATA14A3-expected_output0] PASSED                                              [  3%]
tests/test_call.py::testparse_md[-expected_output1] PASSED                                                       [  4%]
tests/test_call.py::testparse_md[10-expected_output2] PASSED                                                     [  4%]
tests/test_call.py::testparse_md[^ACGT-expected_output3] PASSED                                                  [  5%]
tests/test_call.py::testparse_md[AGCT-expected_output4] PASSED                                                   [  6%]
tests/test_call.py::testparse_md[5^AC5T2-expected_output5] PASSED                                                [  6%]
tests/test_call.py::testparse_md[1A1^T1-expected_output6] PASSED                                                 [  7%]
tests/test_call.py::testparse_md[100-expected_output7] PASSED                                                    [  8%]
tests/test_call.py::testparse_md[^ACGTACGT-expected_output8] PASSED                                              [  8%]
tests/test_call.py::testparse_md[1ATGC1-expected_output9] PASSED                                                 [  9%]
tests/test_call.py::testjoin_cigar[cigar_tuples0-5H10M5S] PASSED                                                 [ 10%]
tests/test_call.py::testjoin_cigar[cigar_tuples1-10M] PASSED                                                     [ 10%]
tests/test_call.py::testjoin_cigar[cigar_tuples2-] PASSED                                                        [ 11%]
tests/test_call.py::testjoin_cigar[cigar_tuples3-5H10M5H] PASSED                                                 [ 12%]
tests/test_call.py::testjoin_cigar[cigar_tuples4-5S10M] PASSED                                                   [ 12%]
tests/test_call.py::testjoin_cigar[cigar_tuples5-10M5S] PASSED                                                   [ 13%]
tests/test_call.py::testjoin_cigar[cigar_tuples6-5H10M5S] PASSED                                                 [ 14%]
tests/test_call.py::testjoin_cigar[cigar_tuples7-5S10M5H] PASSED                                                 [ 14%]
tests/test_call.py::test_trim_clips[5S10M3S-AAAAATTTTTGGGGGCCC-10M-TTTTTGGGGG] PASSED                            [ 15%]
tests/test_call.py::test_trim_clips[10M-TTTTTTTTTT-10M-TTTTTTTTTT] PASSED                                        [ 16%]
tests/test_call.py::test_trim_clips[5H10M5H-TTTTTTTTTT-10M-TTTTTTTTTT] PASSED                                    [ 16%]
tests/test_call.py::test_trim_clips[5S10M-AAAAATTTTTTTTTT-10M-TTTTTTTTTT] PASSED                                 [ 17%]
tests/test_call.py::test_trim_clips[10M3S-TTTTTTTTTTGGG-10M-TTTTTTTTTT] PASSED                                   [ 18%]
tests/test_call.py::test_trim_clips[5S10M5H-AAAAATTTTTTTTTT-10M-TTTTTTTTTT] PASSED                               [ 18%]
tests/test_call.py::test_trim_clips[5H10M5S-TTTTTTTTTTGGGGG-10M-TTTTTTTTTT] PASSED                               [ 19%]
tests/test_call.py::test_generate_cs_tag_long_form[8M2D4M2I-8^AG6-ACGTACGTACGTAC-=ACGTACGT-ag=ACGT+ac] PASSED    [ 20%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-5-ACGTA-=ACGTA] PASSED                                     [ 20%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-3C1-ACGTG-=ACG*ct=G] PASSED                                [ 21%]
tests/test_call.py::test_generate_cs_tag_long_form[5M1I3M-9-ACGTAGCTA-=ACGTA+g=CTA] PASSED                       [ 22%]
tests/test_call.py::test_generate_cs_tag_long_form[5M1D4M-5^A4-ACGTGGCTA-=ACGTG-a=GCTA] PASSED                   [ 22%]
tests/test_call.py::test_generate_cs_tag_long_form[3M1D1M1D4M-3^C1^A4-ACGGCTAG-=ACG-c=G-a=CTAG] PASSED           [ 23%]
tests/test_call.py::test_generate_cs_tag_long_form[3S5M-5-NNNACGTA-=ACGTA] PASSED                                [ 24%]
tests/test_call.py::test_generate_cs_tag_long_form[8M2D4M2I3N1M-2A5^AG7-ACGTACGTACGTACG-=AC*ag=TACGT-ag=ACGT+ac~nn3nn=G] PASSED [ 24%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-0C4-ACGTA-*ca=CGTA] PASSED                                 [ 25%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-4C0-ACGTA-=ACGT*ca] PASSED                                 [ 26%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-0C3C0-ACGTA-*ca=CGT*ca0] PASSED                            [ 26%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-0C3C0-ACGTA-*ca=CGT*ca1] PASSED                            [ 27%]
tests/test_call.py::test_generate_cs_tag_long_form[5M-2CC1-ACGTA-=AC*cg*ct=A] PASSED                             [ 28%]
tests/test_call.py::test_generate_cs_tag_long_form[3M2D1M2I1M3N1M-3^GG0A2-AAATCCTT-=AAA-gg*at+cc=T~nn3nn=T] PASSED [ 28%]
tests/test_call.py::test_generate_cs_tag_short_form[8M2D4M2I-8^AG6-ACGTACGTACGTAC-:8-ag:4+ac] PASSED             [ 29%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-5-ACGTA-:5] PASSED                                        [ 30%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-3C1-ACGTG-:3*ct:1] PASSED                                 [ 30%]
tests/test_call.py::test_generate_cs_tag_short_form[5M1I3M-9-ACGTAGCTA-:5+g:3] PASSED                            [ 31%]
tests/test_call.py::test_generate_cs_tag_short_form[5M1D4M-5^A4-ACGTGGCTA-:5-a:4] PASSED                         [ 32%]
tests/test_call.py::test_generate_cs_tag_short_form[3M1D1M1D4M-3^C1^A4-ACGGCTAG-:3-c:1-a:4] PASSED               [ 32%]
tests/test_call.py::test_generate_cs_tag_short_form[3S5M-5-NNNACGTA-:5] PASSED                                   [ 33%]
tests/test_call.py::test_generate_cs_tag_short_form[8M2D4M2I3N1M-2A5^AG7-ACGTACGTACGTACG-:2*ag:5-ag:4+ac~nn3nn:1] PASSED [ 34%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-0C3C0-ACGTA-*ca:3*ca] PASSED                              [ 34%]
tests/test_call.py::test_generate_cs_tag_short_form[5M-2CC1-ACGTA-:2*cg*ct:1] PASSED                             [ 35%]
tests/test_call.py::test_generate_cs_tag_short_form[3M2D1M2I1M3N1M-3^GG0A2-AAATCCTT-:3-gg*at+cc:1~nn3nn:1] PASSED [ 36%]
tests/test_consensus.py::test_split_cs_tags PASSED                                                               [ 36%]
tests/test_consensus.py::test_normalize_read_lengths PASSED                                                      [ 37%]
tests/test_consensus.py::test_get_consensus PASSED                                                               [ 38%]
tests/test_consensus.py::test_substitution PASSED                                                                [ 38%]
tests/test_consensus.py::test_insertion PASSED                                                                   [ 39%]
tests/test_consensus.py::test_deletion PASSED                                                                    [ 40%]
tests/test_consensus.py::test_splicing PASSED                                                                    [ 40%]
tests/test_consensus.py::test_positions PASSED                                                                   [ 41%]
tests/test_consensus.py::test_positions_more_than_one PASSED                                                     [ 42%]
tests/test_lengthen.py::test_mutation PASSED                                                                     [ 42%]
tests/test_lengthen.py::test_softclip PASSED                                                                     [ 43%]
tests/test_lengthen.py::test_substitution PASSED                                                                 [ 44%]
tests/test_lengthen.py::test_deletion PASSED                                                                     [ 44%]
tests/test_lengthen.py::test_insertion PASSED                                                                    [ 45%]
tests/test_lengthen.py::test_splicing PASSED                                                                     [ 46%]
tests/test_lengthen.py::test_5bpIns_3bp_Del PASSED                                                               [ 46%]
tests/test_lengthen.py::test_first_5nt_del_softclip_5nt PASSED                                                   [ 47%]
tests/test_lengthen.py::test_softclip_plus_5nt PASSED                                                            [ 48%]
tests/test_lengthen.py::test_softclip_plusminus_10nt PASSED                                                      [ 48%]
tests/test_lengthen.py::test_real PASSED                                                                         [ 49%]
tests/test_mask.py::test_basic PASSED                                                                            [ 50%]
tests/test_mask.py::test_basic_with_prefix PASSED                                                                [ 51%]
tests/test_mask.py::test_all_n PASSED                                                                            [ 51%]
tests/test_mask.py::test_softclip PASSED                                                                         [ 52%]
tests/test_mask.py::test_splicing PASSED                                                                         [ 53%]
tests/test_mask.py::test_threshold_2 PASSED                                                                      [ 53%]
tests/test_mask.py::test_threshold_15 PASSED                                                                     [ 54%]
tests/test_mask.py::test_error_cstag_short PASSED                                                                [ 55%]
tests/test_mask.py::test_error_threshold_float PASSED                                                            [ 55%]
tests/test_mask.py::test_error_threshold_45 PASSED                                                               [ 56%]
tests/test_revcomp.py::test_revcomp_normal PASSED                                                                [ 57%]
tests/test_revcomp.py::test_revcomp_prefix PASSED                                                                [ 57%]
tests/test_revcomp.py::test_revcomp_empty PASSED                                                                 [ 58%]
tests/test_revcomp.py::test_revcomp_special_chars PASSED                                                         [ 59%]
tests/test_revcomp.py::test_revcomp_invalid PASSED                                                               [ 59%]
tests/test_shorten.py::test_mutation PASSED                                                                      [ 60%]
tests/test_shorten.py::test_softclip PASSED                                                                      [ 61%]
tests/test_shorten.py::test_substitution PASSED                                                                  [ 61%]
tests/test_shorten.py::test_deletion PASSED                                                                      [ 62%]
tests/test_shorten.py::test_insertion PASSED                                                                     [ 63%]
tests/test_shorten.py::test_splicing PASSED                                                                      [ 63%]
tests/test_shorten.py::test_5bpIns_3bp_Del PASSED                                                                [ 64%]
tests/test_shorten.py::test_first_5nt_del_softclip_5nt PASSED                                                    [ 65%]
tests/test_shorten.py::test_softclip_plus_5nt PASSED                                                             [ 65%]
tests/test_shorten.py::test_softclip_plusminus_10nt PASSED                                                       [ 66%]
tests/test_shorten.py::test_real PASSED                                                                          [ 67%]
tests/test_split.py::test_split[-False-expected_output0] PASSED                                                  [ 67%]
tests/test_split.py::test_split[cs:Z:-False-expected_output1] PASSED                                             [ 68%]
tests/test_split.py::test_split[cs:Z:-True-expected_output2] PASSED                                              [ 69%]
tests/test_split.py::test_split[:4-False-expected_output3] PASSED                                                [ 69%]
tests/test_split.py::test_split[:4:3-False-expected_output4] PASSED                                              [ 70%]
tests/test_split.py::test_split[:4*ag:3-False-expected_output5] PASSED                                           [ 71%]
tests/test_split.py::test_split[:4+ag:3-False-expected_output6] PASSED                                           [ 71%]
tests/test_split.py::test_split[:4-ag:3-False-expected_output7] PASSED                                           [ 72%]
tests/test_split.py::test_split[:4~gt1ac:3-False-expected_output8] PASSED                                        [ 73%]
tests/test_split.py::test_split[:2*ag+t-ccc~gt1ac:2-False-expected_output9] PASSED                               [ 73%]
tests/test_split.py::test_split[=ACGT-False-expected_output10] PASSED                                            [ 74%]
tests/test_split.py::test_split[=ACGT=CGTA-False-expected_output11] PASSED                                       [ 75%]
tests/test_split.py::test_split[=ACGT*ag=CGT-False-expected_output12] PASSED                                     [ 75%]
tests/test_split.py::test_split[=ACGT+ag=CGT-False-expected_output13] PASSED                                     [ 76%]
tests/test_split.py::test_split[=ACGT-ag=CGT-False-expected_output14] PASSED                                     [ 77%]
tests/test_split.py::test_split[=ACGT~gt1ac=CGT-False-expected_output15] PASSED                                  [ 77%]
tests/test_split.py::test_split[=ACGT*ac+gg-cc=T-False-expected_output16] PASSED                                 [ 78%]
tests/test_split.py::test_split[=AC*ag+t-ccc~gt1ac=AC-False-expected_output17] PASSED                            [ 79%]
tests/test_to_html.py::test_append_mark_to_n PASSED                                                              [ 79%]
tests/test_to_html.py::test_split_cstag PASSED                                                                   [ 80%]
tests/test_to_html.py::test_html PASSED                                                                          [ 81%]
tests/test_to_html.py::test_html_repeat_substitution PASSED                                                      [ 81%]
tests/test_to_html.py::test_html_repeat_substitution_start PASSED                                                [ 82%]
tests/test_to_html.py::test_html_repeat_substitution_end PASSED                                                  [ 83%]
tests/test_to_html.py::test_html_start_from_N PASSED                                                             [ 83%]
tests/test_to_html.py::test_html_deletion_with_N PASSED                                                          [ 84%]
tests/test_to_html.py::test_html_N_within_deletions PASSED                                                       [ 85%]
tests/test_to_html.py::test_html_N_within_insertions PASSED                                                      [ 85%]
tests/test_to_sequence.py::test_to_sequence_normal_cases PASSED                                                  [ 86%]
tests/test_to_sequence.py::test_to_sequence_edge_cases PASSED                                                    [ 87%]
tests/test_to_vcf.py::test_find_ref_for_insertion PASSED                                                         [ 87%]
tests/test_to_vcf.py::test_find_ref_for_deletion PASSED                                                          [ 88%]
tests/test_to_vcf.py::test_get_variant_annotations PASSED                                                        [ 89%]
tests/test_to_vcf.py::test_get_pos_end PASSED                                                                    [ 89%]
tests/test_to_vcf.py::test_format_cs_tags PASSED                                                                 [ 90%]
tests/test_to_vcf.py::test_group_by_chrom PASSED                                                                 [ 91%]
tests/test_to_vcf.py::test_group_by_overlapping_intervals PASSED                                                 [ 91%]
tests/test_to_vcf.py::test_call_reference_depth PASSED                                                           [ 92%]
tests/test_to_vcf.py::test_add_vcf_fields PASSED                                                                 [ 93%]
tests/test_to_vcf.py::test_process_cs_tag PASSED                                                                 [ 93%]
tests/test_to_vcf.py::test_process_cs_tags_simple_case PASSED                                                    [ 94%]
tests/test_to_vcf.py::test_process_cs_tags_with_splice PASSED                                                    [ 95%]
tests/test_validator.py::test_validate_cs_tag_normal_cases PASSED                                                [ 95%]
tests/test_validator.py::test_validate_cs_tag_abnormal_cases PASSED                                              [ 96%]
tests/test_validator.py::test_validate_cs_tag_edge_cases PASSED                                                  [ 97%]
tests/test_validator.py::test_validate_short_format PASSED                                                       [ 97%]
tests/test_validator.py::test_validate_long_format PASSED                                                        [ 98%]
tests/test_validator.py::test_validate_threshold PASSED                                                          [ 99%]
tests/test_validator.py::test_validate_pos PASSED                                                                [100%]

---------- coverage: platform linux, python 3.12.0-final-0 -----------
Name                           Stmts   Miss  Cover
--------------------------------------------------
src/cstag/__init__.py             11      0   100%
src/cstag/call.py                120      3    98%
src/cstag/consensus.py            75      4    95%
src/cstag/revcomp.py              27      0   100%
src/cstag/split.py                 9      0   100%
src/cstag/to_html.py              54      0   100%
src/cstag/to_vcf.py              174      8    95%
src/cstag/utils/validator.py      20      0   100%
tests/__init__.py                  0      0   100%
tests/test_call.py                24      0   100%
tests/test_consensus.py           45      0   100%
tests/test_lengthen.py            83      0   100%
tests/test_mask.py                63      3    95%
tests/test_revcomp.py             22      0   100%
tests/test_shorten.py             64      0   100%
tests/test_split.py                5      0   100%
tests/test_to_html.py             94      0   100%
tests/test_to_sequence.py         15      0   100%
tests/test_to_vcf.py              92      0   100%
tests/test_validator.py           78     10    87%
--------------------------------------------------
TOTAL                           1075     28    97%


================================================= 149 passed in 1.94s ==================================================
$ cd ../cstag-cli
$ pytest -vv --cov tests/
================================================= test session starts ==================================================
platform linux -- Python 3.12.0, pytest-7.4.3, pluggy-1.3.0 -- /<PATH>/miniconda3/envs/joss_cstag_test/bin/python3.12
cachedir: .pytest_cache
rootdir: /<PATH>/Downloads/cstag/cstag-cli
plugins: cov-4.1.0
collected 2 items

tests/utils/test_io.py::test_determine_format PASSED                                                             [ 50%]
tests/utils/test_io.py::test_read_sam_with_sam_file PASSED                                                       [100%]

---------- coverage: platform linux, python 3.12.0-final-0 -----------
Name                     Stmts   Miss  Cover
--------------------------------------------
tests/utils/test_io.py      14      0   100%
--------------------------------------------
TOTAL                       14      0   100%


================================================== 2 passed in 0.10s ===================================================
```

* :heavy_check_mark: `cstag` has 97% code coverage
* :heavy_check_mark: `cstag-cli` has 100% code coverage

---
## Testing doctests

Because I saw docstrings, I figured there might be doctests. So I ran pytest on your docstrings and found some errors:

```bash
$ cd ../cstag/src/cstag
$ pytest -vv --doctest-modules
================================================= test session starts ==================================================
platform linux -- Python 3.12.0, pytest-7.4.3, pluggy-1.3.0 -- /<PATH>/miniconda3/envs/joss_cstag_test/bin/python3.12
cachedir: .pytest_cache
rootdir: /<PATH>/Downloads/cstag/cstag/src/cstag
plugins: cov-4.1.0
collected 11 items

call.py::cstag.call.call PASSED                                                                                  [  9%]
consensus.py::cstag.consensus.consensus FAILED                                                                   [ 18%]
lengthen.py::cstag.lengthen.lengthen FAILED                                                                      [ 27%]
mask.py::cstag.mask.mask FAILED                                                                                  [ 36%]
revcomp.py::cstag.revcomp.revcomp FAILED                                                                         [ 45%]
shorten.py::cstag.shorten.shorten FAILED                                                                         [ 54%]
split.py::cstag.split.split FAILED                                                                               [ 63%]
to_html.py::cstag.to_html.to_html FAILED                                                                         [ 72%]
to_pdf.py::cstag.to_pdf.to_pdf FAILED                                                                            [ 81%]
to_sequence.py::cstag.to_sequence.to_sequence FAILED                                                             [ 90%]
to_vcf.py::cstag.to_vcf.to_vcf FAILED                                                                            [100%]

======================================================= FAILURES =======================================================
_________________________________________ [doctest] cstag.consensus.consensus __________________________________________
139         cs_tags (list): CS tags in the **long** format
140         positions (list): 1-based leftmost mapping position (4th column in SAM file)
141         prefix (bool, optional): Whether to add the prefix 'cs:Z:' to the CS tag. Defaults to False
142     Return:
143         str: a consensus of CS tag in the **long** format
144     Example:
145         >>> import cstag
146         >>> cs_tags = ["=ACGT", "=AC*gt=T", "=C*gt=T", "=C*gt=T", "=ACT+ccc=T"]
147         >>> positions = [1,1,1,2,1]
148         >>> cstag.consensus(cs_tags, positions)
Expected:
    =AC*gt=T
Got:
    '=AC*gt=T'

/<PATH>/Downloads/cstag/cstag/src/cstag/consensus.py:148: DocTestFailure
__________________________________________ [doctest] cstag.lengthen.lengthen ___________________________________________
014
015     Return:
016         str: cs tag in **long** form
017
018     Example:
019         >>> import cstag
020         >>> cs = ":4*ag:3"
021         >>> cigar = "8M"
022         >>> seq = "ACGTACGT"
023         >>> cstag.lengthen(cs, cigar, seq)
Expected:
    =ACGT*ag=CGT
Got:
    '=ACGT*ag=CGT'

/<PATH>/Downloads/cstag/cstag/src/cstag/lengthen.py:23: DocTestFailure
______________________________________________ [doctest] cstag.mask.mask _______________________________________________
014         threshold (int, optional): Phred Quality Score (defalt = 10). The low-quality bases are defined as 'less than or equal to the threshold'
015         prefix (bool, optional): Whether to add the prefix 'cs:Z:' to the cs tag. Defaults to False
016     Return:
017         str: Masked cs tag
018     Example:
019         >>> import cstag
020         >>> cs_tag = "=ACGT*ac+gg-cc=T"
021         >>> cigar = "5M2I2D1M"
022         >>> qual = "AA!!!!AA"
023         >>> cstag.mask(cs_tag, qual)
UNEXPECTED EXCEPTION: TypeError("mask() missing 1 required positional argument: 'qual'")
Traceback (most recent call last):
  File "/<PATH>/miniconda3/envs/joss_cstag_test/lib/python3.12/doctest.py", line 1357, in __run
    exec(compile(example.source, filename, "single",
  File "<doctest cstag.mask.mask[4]>", line 1, in <module>
TypeError: mask() missing 1 required positional argument: 'qual'
/<PATH>/Downloads/cstag/cstag/src/cstag/mask.py:23: UnexpectedException
___________________________________________ [doctest] cstag.revcomp.revcomp ____________________________________________
028         cs_tag (str): a cs tag
029         prefix (bool, optional): Whether to add the prefix 'cs:Z:' to the cs tag. Defaults to False
030
031     Return:
032         str: reverse complement of a cs tag
033
034     Example:
035         >>> import cstag
036         >>> cs = "=AAAA*ag=CTG"
037         >>> cstag.revcomp(cs)
Expected:
    =CAG*tc=TTTT
Got:
    '=CAG*tc=TTTT'

/<PATH>/Downloads/cstag/cstag/src/cstag/revcomp.py:37: DocTestFailure
___________________________________________ [doctest] cstag.shorten.shorten ____________________________________________
007 Convert long format of cs tag into short format
008     Args:
009         cs_tag (str): cs tag in the **long** format
010         prefix (bool, optional): Whether to add the prefix 'cs:Z:' to the cs tag. Defaults to False
011     Return:
012         str: cs tag in the **short** format
013     Example:
014         >>> import cstag
015         >>> cs = "=ACGT*ag=CGT"
016         >>> cstag.shorten(cs, prefix=True)
Expected:
    cs:Z::4*ag:3
Got:
    'cs:Z::4*ag:3'

/<PATH>/Downloads/cstag/cstag/src/cstag/shorten.py:16: DocTestFailure
_____________________________________________ [doctest] cstag.split.split ______________________________________________
008     Args:
009         cs_tag (str): a cs tag
010         prefix (bool, optional): Whether to add the prefix 'cs:Z:' to the cs tag. Defaults to False
011     Return:
012         list[str]: splits a cs tag by operators
013
014     Example:
015         >>> import cstag
016         >>> cs = ":4*ag:3"
017         >>> cstag.split(cs)
Expected:
    [":4", "*ag", ":3"]
Got:
    [':4', '*ag', ':3']

/<PATH>/Downloads/cstag/cstag/src/cstag/split.py:17: DocTestFailure
___________________________________________ [doctest] cstag.to_html.to_html ____________________________________________
140     Args:
141         cs_tag (str): CS tag in the **long** format
142         description (str): (optional) header information in the output string
143     Return:
144         HTML string
145     Example:
146         >>> import cstag
147         >>> cs_tag = "=AC+GGG=T-ACGT*at~gt10cg=GNNN"
148         >>> description = "Example"
149         >>> html_string = cstag.to_html(cs_tag, description)
UNEXPECTED EXCEPTION: ValueError('Invalid CS tag: =AC+GGG=T-ACGT*at~gt10cg=GNNN')
Traceback (most recent call last):
  File "/<PATH>/miniconda3/envs/joss_cstag_test/lib/python3.12/doctest.py", line 1357, in __run
    exec(compile(example.source, filename, "single",
  File "<doctest cstag.to_html.to_html[3]>", line 1, in <module>
  File "/<PATH>/Downloads/cstag/cstag/src/cstag/to_html.py", line 151, in to_html
    validate_cs_tag(cs_tag)
  File "/<PATH>/Downloads/cstag/cstag/src/cstag/utils/validator.py", line 12, in validate_cs_tag
    raise ValueError(f"Invalid CS tag: {cs_tag}")
ValueError: Invalid CS tag: =AC+GGG=T-ACGT*at~gt10cg=GNNN
/<PATH>/Downloads/cstag/cstag/src/cstag/to_html.py:149: UnexpectedException
____________________________________________ [doctest] cstag.to_pdf.to_pdf _____________________________________________
020
021     Returns:
022         None: This function does not return anything.
023
024     Examples:
025         >>> import cstag
026         >>> cs_tag = "=AC+GGG=T-ACGT*at~gt10cg=GNNN"
027         >>> description = "Example"
028         >>> path_out = "report.pdf"
029         >>> to_pdf(cs_tag, description, path_out)
UNEXPECTED EXCEPTION: ValueError('Invalid CS tag: =AC+GGG=T-ACGT*at~gt10cg=GNNN')
Traceback (most recent call last):
  File "/<PATH>/miniconda3/envs/joss_cstag_test/lib/python3.12/doctest.py", line 1357, in __run
    exec(compile(example.source, filename, "single",
  File "<doctest cstag.to_pdf.to_pdf[4]>", line 1, in <module>
  File "/<PATH>/Downloads/cstag/cstag/src/cstag/to_pdf.py", line 31, in to_pdf
    cs_tag_html = to_html(cs_tag, description)
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/<PATH>/Downloads/cstag/cstag/src/cstag/to_html.py", line 151, in to_html
    validate_cs_tag(cs_tag)
  File "/<PATH>/Downloads/cstag/cstag/src/cstag/utils/validator.py", line 12, in validate_cs_tag
    raise ValueError(f"Invalid CS tag: {cs_tag}")
ValueError: Invalid CS tag: =AC+GGG=T-ACGT*at~gt10cg=GNNN
/<PATH>/Downloads/cstag/cstag/src/cstag/to_pdf.py:29: UnexpectedException
_______________________________________ [doctest] cstag.to_sequence.to_sequence ________________________________________
010     Args:
011         cs_tag (str): CS tag in the **long** format
012
013     Returns:
014         str: The sequence string derived from the CS tag.
015
016     Example:
017         >>> import cstag
018         >>> cs_tag = "=AC*gt=T-gg=C+tt=A"
019         >>> cstag.to_sequence(cs_tag)
Expected:
    "ACTTCTTA"
Got:
    'ACTTCTTA'

/<PATH>/Downloads/cstag/cstag/src/cstag/to_sequence.py:19: DocTestFailure
____________________________________________ [doctest] cstag.to_vcf.to_vcf _____________________________________________
327         pos (int | list[int]): The starting position for the sequence.
328
329     Returns:
330         str: The VCF-formatted string.
331     Example:
332         >>> import cstag
333         >>> cs_tag = "=AC*gt=T-gg=C+tt=A"
334         >>> chrom = "chr1"
335         >>> pos = 1
336         >>> print(cstag.to_vcf(cstag, chrom, pos))
UNEXPECTED EXCEPTION: TypeError("cs_tags must be str or list, not <class 'module'>")
Traceback (most recent call last):
  File "/<PATH>/miniconda3/envs/joss_cstag_test/lib/python3.12/doctest.py", line 1357, in __run
    exec(compile(example.source, filename, "single",
  File "<doctest cstag.to_vcf.to_vcf[4]>", line 1, in <module>
  File "/<PATH>/Downloads/cstag/cstag/src/cstag/to_vcf.py", line 348, in to_vcf
    raise TypeError(f"cs_tags must be str or list, not {type(cs_tags)}")
TypeError: cs_tags must be str or list, not <class 'module'>
/<PATH>/Downloads/cstag/cstag/src/cstag/to_vcf.py:336: UnexpectedException
=============================================== short test summary info ================================================
FAILED consensus.py::cstag.consensus.consensus
FAILED lengthen.py::cstag.lengthen.lengthen
FAILED mask.py::cstag.mask.mask
FAILED revcomp.py::cstag.revcomp.revcomp
FAILED shorten.py::cstag.shorten.shorten
FAILED split.py::cstag.split.split
FAILED to_html.py::cstag.to_html.to_html
FAILED to_pdf.py::cstag.to_pdf.to_pdf
FAILED to_sequence.py::cstag.to_sequence.to_sequence
FAILED to_vcf.py::cstag.to_vcf.to_vcf
============================================= 10 failed, 1 passed in 0.89s =============================================
```
* :triangular_flag_on_post: It appears that a majority of your doctest errors are purely syntax errors (e.g., `cstag.to_vcf` doctest example makes `cs_tag` variable, but tests for `cstags` instead).

---
# `README.md`
Running the [Usage](https://github.com/akikuno/cstag/tree/main#-usage) as written.
```bash
$ conda activate joss_cstag
$ python
Python 3.10.8 | packaged by conda-forge | (main, Nov 22 2022, 08:23:14) [GCC 10.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import cstag
>>> cigar = "8M2D4M2I3N1M"
>>> md = "2A5^AG7"
>>> seq = "ACGTACGTACGTACG"
>>> print(cstag.call(cigar, md, seq))
:2*ag:5-ag:4+ac~nn3nn:1
>>> print(cstag.call(cigar, md, seq, long=True))
=AC*ag=TACGT-ag=ACGT+ac~nn3nn=G
>>> cs_tag = "=ACGT*ag=CGT"
>>> print(cstag.shorten(cs_tag))
:4*ag:3
>>> cs_tag = ":4*ag:3"
>>> cigar = "8M"
>>> seq = "ACGTACGT"
>>> print(cstag.lengthen(cs_tag, cigar, seq))
=ACGT*ag=CGT
>>> cs_tags = ["=ACGT", "=AC*gt=T", "=C*gt=T", "=C*gt=T", "=ACT+ccc=T"]
>>> positions = [1, 1, 2, 2, 1]
>>> print(cstag.consensus(cs_tags, positions))
=AC*gt=T
>>> cs_tag = "=ACGT*ac+gg-cc=T"
>>> cigar = "5M2I2D1M"
>>> qual = "AA!!!!AA"
>>> phred_threshold = 10
>>> print(cstag.mask(cs_tag, cigar, qual, phred_threshold))
=ACNN*an+ng-cc=T
>>> cs_tag = "=ACGT*ac+gg-cc=T"
>>> print(cstag.split(cs_tag))
['=ACGT', '*ac', '+gg', '-cc', '=T']
>>> cs_tag = "=ACGT*ac+gg-cc=T"
>>> print(cstag.revcomp(cs_tag))
=A-gg+cc*tg=ACGT
>>> cs_tag = "=AC*gt=T-gg=C+tt=A"
>>> print(cstag.to_sequence(cs_tag))
ACTTCTTA
>>> cs_tag = "=AC*gt=T-gg=C+tt=A"
>>> chrom = "chr1"
>>> pos = 1
>>> print(cstag.to_vcf(cs_tag, chrom, pos))
##fileformat=VCFv4.2
#CHROM  POS ID  REF ALT QUAL    FILTER  INFO
chr1    3   .   G   T   .   .   .
chr1    4   .   TGG T   .   .   .
chr1    5   .   C   CTT .   .   .
>>> cs_tags = ["=ACGT", "=AC*gt=T", "=C*gt=T", "=ACGT", "=AC*gt=T"]
>>> chroms = ["chr1", "chr1", "chr1", "chr2", "chr2"]
>>> positions = [2, 2, 3, 10, 100]
>>> print(cstag.to_vcf(cs_tags, chroms, positions))
##fileformat=VCFv4.2
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total Depth">
##INFO=<ID=RD,Number=1,Type=Integer,Description="Depth of Ref allele">
##INFO=<ID=AD,Number=1,Type=Integer,Description="Depth of Alt allele">
##INFO=<ID=VAF,Number=1,Type=Float,Description="Variant allele frequency (AD/DP)">
#CHROM  POS ID  REF ALT QUAL    FILTER  INFO
chr1    4   .   G   T   .   .   DP=3;RD=1;AD=2;VAF=0.667
chr2    102 .   G   T   .   .   DP=1;RD=0;AD=1;VAF=1.0
>>> from pathlib import Path
>>> cs_tag = "=AC+ggg=T-acgt*at~gt10ag=GNNN"
>>> description = "Example"
>>> cs_tag_html = cstag.to_html(cs_tag, description)
>>> Path("report.html").write_text(cs_tag_html)
1539
>>> cs_tag = "=AC+ggg=T-acgt*at~gt10ag=GNNN"
>>> description = "Example"
>>> path_out = "report.pdf"
>>> cstag.to_pdf(cs_tag, description, path_out)
>>> quit()
$ cd ~/Downloads/cstag/cstag-cli
$ cstag append tests/append/data/example.bam > example_cs_short.sam
$ ll
total 28K
drwxr-xr-x 1 root root 4.0K Nov 20 06:25 dist/
drwxr-xr-x 1 root root 4.0K Nov 20 05:50 src/
drwxr-xr-x 1 root root 4.0K Nov 20 05:50 tests/
-rw-r--r-- 1 root root 5.1K Nov 20 05:50 CODE_OF_CONDUCT.md
-rw-r--r-- 1 root root 7.9K Nov 20 05:50 CONTRIBUTING.md
-rw-r--r-- 1 root root 1.1K Nov 20 05:50 LICENSE
-rw-r--r-- 1 root root 2.5K Nov 20 05:50 README.md
-rw-r--r-- 1 root root  342 Nov 20 08:13 example_cs_short.sam
-rw-r--r-- 1 root root  861 Nov 20 05:50 pyproject.toml
$ head example_cs_short.sam
@HD VN:1.3  SO:coordinate
@SQ SN:chr1 LN:1000
@SQ SN:chr2 LN:1000
read1   0   chr1    500 25  18M2D6M1I11M    *   0   0   AAAAAAAAAAAAAAAAAAATTTTTAAAATTTTTAAA    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!    MD:Z:18^AG17    cs:Z::18-ag:6+a:11
read2   16  chr2    500 25  36M *   0   0   ATATAACAAACCCTGAGAACCAAAATGAACGAAAAC    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!MD:Z:0C34T0 cs:Z:*ca:34*tc
$ cstag append tests/append/data/example.bam --long > example_cs_long.sam
$ ll
...
-rw-r--r-- 1 root root  404 Nov 20 08:14 example_cs_long.sam
-rw-r--r-- 1 root root  342 Nov 20 08:13 example_cs_short.sam
...
$ head example_cs_long.sam
@HD VN:1.3  SO:coordinate
@SQ SN:chr1 LN:1000
@SQ SN:chr2 LN:1000
read1   0   chr1    500 25  18M2D6M1I11M    *   0   0   AAAAAAAAAAAAAAAAAAATTTTTAAAATTTTTAAA    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!    MD:Z:18^AG17    cs:Z:=AAAAAAAAAAAAAAAAAA-ag=ATTTTT+a=AAATTTTTAAA
read2   16  chr2    500 25  36M *   0   0   ATATAACAAACCCTGAGAACCAAAATGAACGAAAAC    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!MD:Z:0C34T0 cs:Z:*ca=TATAACAAACCCTGAGAACCAAAATGAACGAAAA*tc
$ cat tests/append/data/example.bam | cstag append > example_cs_stdin.sam
[E::idx_find_and_load] Could not retrieve index file for '<stdin>'
$ ll
...
-rw-r--r-- 1 root root  404 Nov 20 08:14 example_cs_long.sam
-rw-r--r-- 1 root root  342 Nov 20 08:17 example_cs_short.sam
-rw-r--r-- 1 root root  342 Nov 20 08:16 example_cs_stdin.sam
...
$ head example_cs_stdin.sam
@HD VN:1.3  SO:coordinate
@SQ SN:chr1 LN:1000
@SQ SN:chr2 LN:1000
read1   0   chr1    500 25  18M2D6M1I11M    *   0   0   AAAAAAAAAAAAAAAAAAATTTTTAAAATTTTTAAA    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!   MD:Z:18^AG17    cs:Z::18-ag:6+a:11
read2   16  chr2    500 25  36M *   0   0   ATATAACAAACCCTGAGAACCAAAATGAACGAAAAC    !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!   MD:Z:0C34T0 cs:Z:*ca:34*tc
```
* :heavy_check_mark: `cstag` usage appears to work as intended.
* :heavy_check_mark: `cstag-cli` usage appears to work as intended
    * :triangular_flag_on_post: Warning from `pysam` when dealing with STDIN regarding missing index file.
    * Given that this warning will always occur when creating a file from STDIN, it may be good to provide either explicit notice to end-users what the error means (i.e., there is no index file for the newly created file
    from STDIN), provide a parameter to suppress the warning, or just have the code fully [workaround](https://github.com/pysam-developers/pysam/issues/939#issuecomment-669016051) the error.

---
# Final notes
* As of this writing (20231120), the build for `cstag` appears to be failing for all [macOS builds](https://github.com/akikuno/cstag/actions/runs/6924735500).
* `cstag-cli` does not appear to have its own build environment and is just reporting `cstag's failing build.
