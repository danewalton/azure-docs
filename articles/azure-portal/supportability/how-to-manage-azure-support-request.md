---
title: Manage an Azure support request
description: Learn about viewing support requests and how to send messages, upload files, and manage options.
tags: billing
ms.topic: how-to
ms.date: 11/02/2021
# To add: close and reopen, review request status, update contact info
---

# Manage an Azure support request

After you [create an Azure support request](how-to-create-azure-support-request.md), you can manage it in the [Azure portal](https://portal.azure.com). You can also create and manage requests programmatically, using the [Azure support ticket REST API](/rest/api/support), or by using [Azure CLI](/cli/azure/azure-cli-support-request).

To manage a support request, you must have the [Owner](../../role-based-access-control/built-in-roles.md#owner), [Contributor](../../role-based-access-control/built-in-roles.md#contributor), or [Support Request Contributor](../../role-based-access-control/built-in-roles.md#support-request-contributor) role at the subscription level. To manage a support request that was created without a subscription, you must be an [Admin](../../active-directory/roles/permissions-reference.md).

## View support requests

View the details and status of support requests by going to **Help + support** >  **All support requests**.

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="All support requests":::

On this page, you can search, filter, and sort support requests. Select a support request to view details, including its severity and any messages associated with the request.

## Send a message

1. On the **All support requests** page, select the support request.

1. On the **Support Request** page, select **New message**.

1. Enter your message and select **Submit**.

## Change the severity level

> [!NOTE]
> The maximum severity level depends on your [support plan](https://azure.microsoft.com/support/plans).

1. On the **All support requests** page, select the support request.

1. On the **Support Request** page, select **Change**.

1. The Azure portal shows one of two screens, depending on whether your request is already assigned to a support engineer:

    - If your request hasn't been assigned, you see a screen like the following. Select a new severity level, then select **Change**.

        :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="Select a new severity level":::

    - If your request has been assigned, you see a screen like the following. Select **OK**, then create a [new message](#send-a-message) to request a change in severity level.

        :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="Can't select a new severity level":::

## Allow collection of advanced diagnostic information

When you create a support request, you can select **Yes** or **No** in the **Advanced diagnostic information** section. This option determines whether Azure support can gather [diagnostic information](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) such as [log files](how-to-create-azure-support-request.md#advanced-diagnostic-information-logs) from your Azure resources that can potentially help resolve your issue.

To change your **Advanced diagnostic information** selection after the request has been created:

1. On the **All support requests** page, select the support request.

1. On the **Support Request** page, look for **Advanced diagnostic information** and then select **Change**.

1. Select **Yes** or **No**, then select **OK** to confirm.

    :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="Grant permissions for diagnostic information":::

## Upload files

You can use the file upload option to upload diagnostic files or any other files that you think are relevant to a support request.

1. On the **All support requests** page, select the support request.

1. On the **Support Request** page, browse to find your file, then select **Upload**. Repeat the process if you have multiple files.

    :::image type="content" source="media/how-to-manage-azure-support-request/file-upload.png" alt-text="Upload file":::

### File upload guidelines

Follow these guidelines when you use the file upload option:

- To protect your privacy, don't include personal information in your upload.
- The file name must be no longer than 110 characters.
- You can't upload more than one file.
- Files can't be larger than 4 MB.
- All files must have a file name extension, such as *.docx* or *.xlsx*. The following table shows the filename extensions that are allowed for upload.

| 0-9, A-C    | D-G   | H-N         | O-Q   | R-T      | U-W        | X-Z     |
|-------------|-------|-------------|-------|----------|------------|---------|
| .7z         | .dat  | .har        | .odx  | .rar     | .uccapilog | .xlam   |
| .a          | .db   | .hwl        | .oft  | .rdl     | .uccplog   | .xlr    |
| .abc        | .DMP  | .ics        | .old  | .rdlc    | .udcx      | .xls    |
| .adm        | .do_  | .ini        | .one  | .re_     | .vb_       | .xlsb   |
| .aspx       | .doc  | .java       | .osd  | .remove  | .vbs_      | .xlsm   |
| .ATF        | .docm | .jpg        | .OUT  | .ren     | .vcf       | .xlsx   |
| .b          | .docx | .LDF        | .p1   | .rename  | .vsd       | .xlt    |
| .ba_        | .dotm | .letterhead | .pcap | .rft     | .wdb       | .xltx   |
| .bak        | .dotx | .lo_        | .pdb  | .rpt     | .wks       | .xml    |
| .blg        | .dtsx | .log        | .pdf  | .rte     | .wma       | .xmla   |
| .CA_        | .eds  | .lpk        | .piz  | .rtf     | .wmv       | .xps    |
| .CAB        | .emf  | .manifest   | .pmls | .run     | .wmz       | .xsd    |
| .cap        | .eml  | .master     | .png  | .saz     | .wps       | .xsn    |
| .catx       | .emz  | .mdmp       | .potx | .sql     | .wpt       | .xxx    |
| .CFG        | .err  | .mof        | .ppt  | .sqlplan | .wsdl      | .z_     |
| .compressed | .etl  | .mp3        | .pptm | .stp     | .wsp       | .z01    |
| .Config     | .evt  | .mpg        | .pptx | .svclog  | .wtl       | .z02    |
| .cpk        | .evtx | .ms_        | .prn  | .tdb     | -          | .zi     |
| .cpp        | .EX   | .msg        | .psf  | .tdf     | -          | .zi_    |
| .cs         | .ex_  | .mso        | .pst  | .text    | -          | .zip    |
| .CSV        | .ex0  | .msu        | .pub  | .thmx    | -          | .zip_   |
| .cvr        | .FRD  | .nfo        | -     | .tif     | -          | .zipp   |
| -           | .gif  | -           | -     | .trc     | -          | .zipped |
| -           | .guid | -           | -     | .TTD     | -          | .zippy  |
| -           | .gz   | -           | -     | .tx_     | -          | .zipx   |
| -           | -     | -           | -     | .txt     | -          | .zit    |
| -           | -     | -           | -     | -        | -          | .zix    |
| -           | -     | -           | -     | -        | -          | .zzz    |

## Close a support request

To close a support request, [send a message](#send-a-message) and let us know you'd like to close the request.

## Reopen a closed request

To reopen a closed support request, create a [new message](#send-a-message), which automatically reopens the request.

## Cancel a support plan

To cancel a support plan, see [Cancel a support plan](../../cost-management-billing/manage/cancel-azure-subscription.md#cancel-a-support-plan).

## Next steps

- Review the process to [create an Azure support request](how-to-create-azure-support-request.md).
- Learn about the [Azure support ticket REST API](/rest/api/support).
