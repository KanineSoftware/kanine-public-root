# Kanine public daily root

This repository is the public copy of the Kanine daily root. Kanine Software writes to it once per UTC date. Kanine
Software cannot change a published line without a visible change in the history of this repository. The `main` branch
refuses a force push and a deletion.

## What the files are

| File | Content |
|---|---|
| `roots.txt` | One root line per UTC date, in date order. Each line ends with one LF. |
| `tokens/YYYY-MM-DD.<authority>.tst` | The RFC 3161 timestamp token of one authority over the root line of that date (DER). |
| `authorities/pinned-roots.pem` | The root certificates that `kanine-verify` pins for the timestamp authorities. |

A root line has this form:

```
public_root spec=kanine.public-root/v1 date=YYYY-MM-DD root=<64 lower-hex> leaves=<n>
```

The root is the RFC 6962 Merkle tree hash over one leaf for each agency. The leaf of an agency is the latest daily
anchor of its audit chain. The root line holds no agency name, no agency id and no record data.

The authorities are DigiCert (`digicert`) and Sectigo (`sectigo`). Each token signs the SHA-256 of the root line, with
no line end.

## How to check an export against this repository

A sealed Kanine export with an audit chain holds a `witness/` folder:

```
witness/YYYY-MM-DD/root.txt
witness/YYYY-MM-DD/inclusion.txt
witness/YYYY-MM-DD/<authority>.tst
```

Do these steps:

1. Run `kanine-verify` on the export. The verifier ships inside the export as source.
2. Make sure that the verifier reports no failure. A bad inclusion path or a bad token gives exit code 2.
3. Read each root line that the verifier prints.
4. Find the line of the same date in `roots.txt` of this repository.
5. Compare the two lines byte for byte.
6. Optionally, compare each `witness/YYYY-MM-DD/<authority>.tst` with `tokens/YYYY-MM-DD.<authority>.tst` here.

If the two root lines are the same, the export agrees with the root that Kanine published for that date. If the lines
differ, or the date is not in `roots.txt`, the export does not agree with the public copy. Report it to Kanine
Software.

You can also check a line with the history of this repository. Run `git log --follow -p roots.txt` to see the commit
that added each line. A published line never changes.

## The format of record

The byte formats are in `PublicRootSpec.cs`, which ships inside each export. The design is in
`app/docs/PUBLIC-DAILY-ROOT.md` of the Kanine source.
