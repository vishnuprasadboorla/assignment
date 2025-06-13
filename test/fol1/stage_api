const resolvedFilePaths = [];
const workspaceFiles = [];

const isUsingFilePaths = Array.isArray(file_paths) && file_paths.length > 0;
const listToProcess = isUsingFilePaths ? file_paths : file_ids;

for (let i = 0; i < listToProcess.length; i++) {
  let file_path = isUsingFilePaths ? listToProcess[i] : undefined;
  let file_id = !isUsingFilePaths ? listToProcess[i] : undefined;

  if (file_path) {
    [, file_path] = await new WorkspaceFiles().resolvePath(
      file_path,
      undefined,
      workspaceBasePath
    ).catch(error => {
      reject(Service.rejectResponse(
        {
          "error_code": error.error_code,
          "error_message": error.error_message || error
        },
        error.status_code || 500
      ));
      throw new Error(error.error_message || error);
    });
  }

  const workspaceFile = file_path
    ? await sequelize.query(
        `SELECT id, lookup_file_types_id, lookup_git_providers_id, git_repo_root_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(:filepath, NULL, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
        {
          replacements: {
            filepath: file_path,
            _workspace_id: workspace[0].id,
            _tenant_id: validToken.tenant_id,
          },
          type: sequelize.QueryTypes.SELECT,
          transaction: t,
          logging: (query, elapsedTime) =>
            Utils.sequelizeLog({
              query,
              elapsedTime,
              requestId: request.requestId,
              serviceLogger: logger,
            }),
        }
      )
    : await sequelize.query(
        `SELECT id, lookup_file_types_id, lookup_git_providers_id, git_repo_root_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(NULL, :fileid, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
        {
          replacements: {
            fileid: file_id,
            _workspace_id: workspace[0].id,
            _tenant_id: validToken.tenant_id,
          },
          type: sequelize.QueryTypes.SELECT,
          transaction: t,
          logging: (query, elapsedTime) =>
            Utils.sequelizeLog({
              query,
              elapsedTime,
              requestId: request.requestId,
              serviceLogger: logger,
            }),
        }
      );

  if (!workspaceFile || workspaceFile.length < 1) {
    reject(Service.rejectResponse(
      {
        error_code: "RFA-000206",
        error_message: `The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
      },
      404
    ));
    throw new Error(`The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
  }

  if (
    workspaceFile[0].lookup_file_types_id === global.lookupFileTypesByName['GIT_FOLDER'] ||
    workspaceFile[0].lookup_file_types_id === global.lookupFileTypesByName['FOLDER']
  ) {
    reject(Service.rejectResponse(
      {
        error: `The provided ${file_id !== undefined ? `File ID: ${file_id}` : `File path: '${file_path}'`} is not a file. Only files can be used to get the file-diff in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
      },
      400
    ));
    throw new Error(`The provided ${file_id !== undefined ? `File ID: ${file_id}` : `File path: '${file_path}'`} is not a file. Only files can be used to get the file-diff in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
  }

  workspaceFiles.push(workspaceFile[0]); // Save to array for later use

  const filePath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
    replacements: {
      fileId: workspaceFile[0]?.id,
      workspaceId: workspace[0].id,
      tenantId: validToken.tenant_id,
    },
    transaction: t,
    logging: (query, elapsedTime) =>
      Utils.sequelizeLog({
        query,
        elapsedTime,
        requestId: request.requestId,
        serviceLogger: logger,
      }),
  });

  if (!filePath[0][0]) {
    reject(Service.rejectResponse(
      {
        error_code: "RFA-000206",
        error_message: `The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
      },
      404
    ));
    throw new Error(`The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
  }

  const filePathToDiscardFile = path.join(
    `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`,
    filePath[0]?.[0]?.path
  );

  resolvedFilePaths.push(filePathToDiscardFile);
}











No staging
file-diff (remove --cached)
status will be 





  /workspace/files/git/stage:
    post:
      security:
        - bearerAuth: []
      description: |
        Stages all changes (new, modified, deleted) in the Git folder located in the specified workspace.
      operationId: stageAllGitFiles
      parameters:
        - in: query
          name: workspace_id
          description: Workspace ID used to locate the Git folder.
          schema:
            type: integer
            format: int64
        - in: query
          name: workspace_name
          description: Workspace name used to locate the Git folder.
          schema:
            type: string
        - in: query
          name: file_id
          description: File ID of the Git folder within the workspace.
          schema:
            type: integer
            format: int64
        - in: query
          name: file_path
          description: Full file path to the Git folder within the workspace.
          schema:
            type: string
            minLength: 1
      responses:
        "200":
          description: Successfully staged all files.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/StageResponse"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthenticated"
        "403":
          $ref: "#/components/responses/PermissionDenied"
        "404":
          $ref: "#/components/responses/NotFound"
        "500":
          $ref: "#/components/responses/Unknown"
      summary: Stage all Git changes in the repo.
      tags:
        - Git
      x-swagger-router-controller: GitController
      x-eov-operation-handler: controllers/GitController

    StageResponse:
      type: object
      properties:
        message:
          type: string
          description: Confirmation message after staging files.
          example: File changes have been successfully staged.


const stageAllFiles = (absoluteRepoPath) => new Promise(
  async (resolve, reject) => {
    const gitRepo = simpleGit(absoluteRepoPath);

    try {
      // Stage all changes (equivalent to `git add .`)
      await gitRepo.add('.');

      resolve({
        message: "All changes have been successfully staged"
      });

    } catch (error) {
      reject(error);
    }
  }
);

// const getFileDiff = (absoluteRepoPath, fullFilePath) => new Promise(async (resolve, reject) => {
//   const gitRepo = simpleGit(absoluteRepoPath);

//   try {
//     // Convert full file path to path relative to the repo (required by Git CLI)
//     const relativePath = path.relative(absoluteRepoPath, fullFilePath);

//     const diffResult = await gitRepo.diff([relativePath]);

//     if (!diffResult.trim()) {
//       return reject({
//         error_message: `No changes found in the selected file.`,
//         status_code: 404
//       });
//     }

//     // Return the diff result
//     resolve(diffResult);

//   } catch (error) {
//     reject(error);
//   }
// });



const stageAllGitFiles = ({ workspace_id, workspace_name, file_id, file_path }, request, response) => new Promise(
  async (resolve, reject) => {
    try {
      logger.info(`Staging all the files of a Git Folder ...`, { requestId: request.requestId });

      const validToken = request.validToken;
      let staging;

      await new Utils.Tenant().validateTenantId(validToken.tenant_id, request.requestId).catch(e => {
        reject(Service.rejectResponse(
          {
            error_code: e.error_code,
            error_message: e.error || e
          },
          e.status || 500
        ));
        throw new Error(`Error while validating Tenant Id: ${e.error || e}`);
      });

      if (workspace_id === undefined && workspace_name === undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide any one of workspace_id or workspace_name to stage the files of a git folder.'
          },
          400
        ));
        return;
      }
      else if (workspace_id !== undefined && workspace_name !== undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide either workspace_id or workspace_name to stage the files of a git folder.'
          },
          400
        ));
        return;
      }

      if (file_id === undefined && file_path === undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide any one of file_id or file_path to stage the files of a git folder.'
          },
          400
        ));
        return;
      }
      else if (file_id !== undefined && file_path !== undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide either file_id or file_path to stage the files of a git folder.'
          },
          400
        ));
        return;
      }

      const [workspace] = await checkWorkspaceExistsAndEnabled({
        workspace_id: workspace_id,
        workspace_name: workspace_name,
        workspace_permission_rfa: {
          permission: ["MANAGE", "EDIT", "RUN"],
          rfa: {
            code: "RFA-000070",
            message: `Insufficient permissions on the provided Workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `Name: '${workspace_name}'`} to stage the files of a git folder.`
          }
        },
        validToken,
        check_enable: undefined,
        request: request
      }).catch(error => {
        reject(Service.rejectResponse(
          {
            "error_code": error.error_code,
            "error_message": error.error_message || error
          },
          error.status_code || 500
        ));
        throw new Error(error.error_message || error);
      });

      const workspaceBasePath = `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`;

      if (file_path !== undefined) {
        [, file_path] = await new WorkspaceFiles().resolvePath(
          file_path,
          undefined,
          workspaceBasePath
        ).catch(error => {
          reject(Service.rejectResponse(
            {
              "error_code": error.error_code,
              "error_message": error.error_message || error
            },
            error.status_code || 500
          ));
          throw new Error(error.error_message || error);
        });
      }

      await config.sequelize.transaction(
        {
          logging: (query, elapsedTime) =>
            Utils.sequelizeLog({
              query,
              elapsedTime,
              requestId: request.requestId,
              serviceLogger: logger,
            }),
        },
        async (t) => {

          const workspaceFile =
            file_path
              ? await sequelize.query(
                `SELECT id, lookup_file_types_id, lookup_git_providers_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(:filepath, NULL, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                {
                  replacements: {
                    filepath: file_path, // Assuming file_path represents the file_path
                    _workspace_id: workspace[0].id,
                    _tenant_id: validToken.tenant_id,
                  },
                  type: sequelize.QueryTypes.SELECT,
                  transaction: t,
                  logging: (query, elapsedTime) =>
                    Utils.sequelizeLog({
                      query,
                      elapsedTime,
                      requestId: request.requestId,
                      serviceLogger: logger,
                    }),
                }
              )
              : await sequelize.query(
                `SELECT id, lookup_file_types_id, lookup_git_providers_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(NULL, :fileid, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                {
                  replacements: {
                    fileid: file_id,
                    _workspace_id: workspace[0].id,
                    _tenant_id: validToken.tenant_id,
                  },
                  transaction: t,
                  type: sequelize.QueryTypes.SELECT, // To fetch rows
                  logging: (query, elapsedTime) =>
                    Utils.sequelizeLog({
                      query,
                      elapsedTime,
                      requestId: request.requestId,
                      serviceLogger: logger,
                    }),
                }
              )

          if (!workspaceFile || workspaceFile.length < 1) {
            reject(Service.rejectResponse(
              {
                error_code: "RFA-000206",
                error_message: `The provided directory ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              404
            ));
            throw new Error(`The provided directory ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }

          if (workspaceFile[0].lookup_file_types_id != global.lookupFileTypesByName['GIT_FOLDER']) {
            reject(Service.rejectResponse(
              {
                error: `The provided  ${file_id !== undefined ? `File ID: ${file_id}` : `File path: '${file_path}'`} is not is not a GIT Repository in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              400
            ));
            throw new Error(`The provided  ${file_id !== undefined ? `File ID: ${file_id}` : `File path: '${file_path}'`} is not is not a GIT Repository in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }


          const filePath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
            replacements: {
              fileId: workspaceFile[0]?.id,
              workspaceId: workspace[0].id,
              tenantId: validToken.tenant_id,
            },
            transaction: t,
            logging: (query, elapsedTime) =>
              Utils.sequelizeLog({
                query,
                elapsedTime,
                requestId: request.requestId,
                serviceLogger: logger,
              }),
          });

          if (!filePath[0][0]) {
            reject(Service.rejectResponse(
              {
                error_code: "RFA-000206",
                error_message: `The provided directory ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              404
            ));
            throw new Error(`The provided directory ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }

          const filePathToGetGitStatus = path.join(`${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`, filePath[0]?.[0]?.path);

          staging = await stageAllFiles(filePathToGetGitStatus).catch(error => {
            reject(Service.rejectResponse(
              {
                "error_code": error.error_code,
                "error_message": error.error_message || error.message || error.error || error
              },
              error.status_code || 500
            ));
            throw new Error(error.error_message || error.message || error.error || error);
          });
        }
      );
      logger.info(`Fetched status of the git folder`, request.requestId);

      resolve(Service.successResponse({
        status: staging
      }));
    } catch (e) {
      logger.error(e.stack || e.message || e.error || e, { requestId: request.requestId });
      reject(Service.rejectResponse(
        { error: e.message || e.stack || JSON.stringify(e) || e, requestId: request.requestId },
        e.status || 500
      ));
    }
  },
);




const stageAllGitFiles = async (request, response) => {
  await Controller.handleRequest(request, response, service.stageAllGitFiles);
};



const discardGitFiles = ({ workspace_id, workspace_name, file_id, file_path }, request, response) => new Promise(
  async (resolve, reject) => {
    try {
      logger.info(`Discarding selected files in a Git Folder ...`, { requestId: request.requestId });

      const validToken = request.validToken;
      let fileDiff;

      await new Utils.Tenant().validateTenantId(validToken.tenant_id, request.requestId).catch(e => {
        reject(Service.rejectResponse(
          {
            error_code: e.error_code,
            error_message: e.error || e
          },
          e.status || 500
        ));
        throw new Error(`Error while validating Tenant Id: ${e.error || e}`);
      });

      if (workspace_id === undefined && workspace_name === undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide any one of workspace_id or workspace_name to discard changes in a file within a Git folder.'
          },
          400
        ));
        return;
      }
      else if (workspace_id !== undefined && workspace_name !== undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide either workspace_id or workspace_name to discard changes in a file within a Git folder.'
          },
          400
        ));
        return;
      }

      if (
        (!Array.isArray(file_id) || file_id.length === 0) &&
        (!Array.isArray(file_path) || file_path.length === 0)
      ) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide at least one of file_id or file_path  to discard changes in a file within a Git folder.'
          },
          400
        ));
        return;
      } else if (
        Array.isArray(file_id) && file_id.length > 0 &&
        Array.isArray(file_path) && file_path.length > 0
      ) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide either file_id or file_path, not both, to discard changes in a file within a Git folder.'
          },
          400
        ));
        return;
      }

      console.log(file_id);
      console.log("-------------------------------------");
      console.log(file_path);

      const [workspace] = await checkWorkspaceExistsAndEnabled({
        workspace_id: workspace_id,
        workspace_name: workspace_name,
        workspace_permission_rfa: {
          permission: ["MANAGE", "EDIT", "RUN"],
          rfa: {
            code: "RFA-000070",
            message: `Insufficient permissions on the provided Workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `Name: '${workspace_name}'`} to discard changes in a file within a Git folder.`
          }
        },
        validToken,
        check_enable: undefined,
        request: request
      }).catch(error => {
        reject(Service.rejectResponse(
          {
            "error_code": error.error_code,
            "error_message": error.error_message || error
          },
          error.status_code || 500
        ));
        throw new Error(error.error_message || error);
      });

      const workspaceBasePath = `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`;

      await config.sequelize.transaction(
        {
          logging: (query, elapsedTime) =>
            Utils.sequelizeLog({
              query,
              elapsedTime,
              requestId: request.requestId,
              serviceLogger: logger,
            }),
        },
        async (t) => {

          if (file_path !== undefined) {
            [, file_path] = await new WorkspaceFiles().resolvePath(
              file_path,
              undefined,
              workspaceBasePath
            ).catch(error => {
              reject(Service.rejectResponse(
                {
                  "error_code": error.error_code,
                  "error_message": error.error_message || error
                },
                error.status_code || 500
              ));
              throw new Error(error.error_message || error);
            });
          }

          const workspaceFile =
            file_path
              ? await sequelize.query(
                `SELECT id, lookup_file_types_id, lookup_git_providers_id, git_repo_root_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(:filepath, NULL, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                {
                  replacements: {
                    filepath: file_path, // Assuming file_path represents the file_path
                    _workspace_id: workspace[0].id,
                    _tenant_id: validToken.tenant_id,
                  },
                  type: sequelize.QueryTypes.SELECT,
                  transaction: t,
                  logging: (query, elapsedTime) =>
                    Utils.sequelizeLog({
                      query,
                      elapsedTime,
                      requestId: request.requestId,
                      serviceLogger: logger,
                    }),
                }
              )
              : await sequelize.query(
                `SELECT id, lookup_file_types_id, lookup_git_providers_id, git_repo_root_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(NULL, :fileid, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                {
                  replacements: {
                    fileid: file_id,
                    _workspace_id: workspace[0].id,
                    _tenant_id: validToken.tenant_id,
                  },
                  transaction: t,
                  type: sequelize.QueryTypes.SELECT, // To fetch rows
                  logging: (query, elapsedTime) =>
                    Utils.sequelizeLog({
                      query,
                      elapsedTime,
                      requestId: request.requestId,
                      serviceLogger: logger,
                    }),
                }
              )

          if (!workspaceFile || workspaceFile.length < 1) {
            reject(Service.rejectResponse(
              {
                error_code: "RFA-000206",
                error_message: `The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              404
            ));
            throw new Error(`The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }

          if (
            workspaceFile[0].lookup_file_types_id === global.lookupFileTypesByName['GIT_FOLDER'] ||
            workspaceFile[0].lookup_file_types_id === global.lookupFileTypesByName['FOLDER']
          ) {
            reject(Service.rejectResponse(
              {
                error: `The provided ${file_id !== undefined ? `File ID: ${file_id}` : `File path: '${file_path}'`} is not a file. Only files can be used to get the file-diff in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              400
            ));

            throw new Error(`The provided ${file_id !== undefined ? `File ID: ${file_id}` : `File path: '${file_path}'`} is not a file. Only files can be used to get the file-diff in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }


          const filePath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
            replacements: {
              fileId: workspaceFile[0]?.id,
              workspaceId: workspace[0].id,
              tenantId: validToken.tenant_id,
            },
            transaction: t,
            logging: (query, elapsedTime) =>
              Utils.sequelizeLog({
                query,
                elapsedTime,
                requestId: request.requestId,
                serviceLogger: logger,
              }),
          });

          if (!filePath[0][0]) {
            reject(Service.rejectResponse(
              {
                error_code: "RFA-000206",
                error_message: `The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              404
            ));
            throw new Error(`The provided file ${file_id !== undefined ? `ID: ${file_id}` : `path: '${file_path}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }

          const filePathToDiscardFile = path.join(`${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`, filePath[0]?.[0]?.path);

          const repoRootId = workspaceFile[0].git_repo_root_id;

          const repoPath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
            replacements: {
              fileId: repoRootId,
              workspaceId: workspace[0].id,
              tenantId: validToken.tenant_id,
            },
            transaction: t,
            logging: (query, elapsedTime) =>
              Utils.sequelizeLog({
                query,
                elapsedTime,
                requestId: request.requestId,
                serviceLogger: logger,
              }),
          });

          if (!repoPath[0][0]) {
            reject(Service.rejectResponse(
              {
                error_code: "RFA-000206",
                error_message: `The provided directory ${repoRootId !== undefined ? `ID: ${repoRootId}` : `path: '${repoPath}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
              },
              404
            ));
            throw new Error(`The provided directory ${repoRootId !== undefined ? `ID: ${repoRootId}` : `path: '${repoPath}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
          }

          const filePathOfRepoRoot = path.join(`${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`, repoPath[0]?.[0]?.path);

          discardFile = await discardMultipleFiles(filePathOfRepoRoot, filePathToDiscardFile).catch(error => {
            reject(Service.rejectResponse(
              {
                "error_code": error.error_code,
                "error_message": error.error_message || error.message || error.error || error
              },
              error.status_code || 500
            ));
            throw new Error(error.error_message || error.message || error.error || error);
          });
        }
      );
      logger.info(`Discarded selected files in a Git Folder.`, request.requestId);

      resolve(Service.successResponse({
        discardFile
      }));
    } catch (e) {
      logger.error(e.stack || e.message || e.error || e, { requestId: request.requestId });
      reject(Service.rejectResponse(
        { error: e.message || e.stack || JSON.stringify(e) || e, requestId: request.requestId },
        e.status || 500
      ));
    }
  },
);


const discardFileChanges = (absoluteRepoPath, fullFilePath) => new Promise(async (resolve, reject) => {
  const gitRepo = simpleGit(absoluteRepoPath);
  const relativePath = path.relative(absoluteRepoPath, fullFilePath);

  try {
    await fs.access(absoluteRepoPath);
    const status = await gitRepo.status();
    const { staged, not_added, renamed } = status;

    const renamedFile = renamed.find(r =>
      r.from === relativePath || r.to === relativePath
    );

    if (renamedFile) {
      const fileToDiscard = renamedFile.from === relativePath ? renamedFile.from : renamedFile.to;
      await gitRepo.reset(["HEAD", fileToDiscard]);

      if (renamedFile.from === relativePath) {
        await gitRepo.checkout(fileToDiscard);
      } else {
        const absolutePathToRemove = path.join(absoluteRepoPath, fileToDiscard);
        await fs.rm(absolutePathToRemove, { force: true });
      }

      resolve();
      return;
    }

    if (staged.includes(relativePath)) {
      await gitRepo.reset(["HEAD", relativePath]);

      const updatedStatus = await gitRepo.status();
      if (updatedStatus.not_added.includes(relativePath)) {
        await fs.rm(fullFilePath, { force: true });
        resolve();
        return;
      }

      await gitRepo.checkout(relativePath);
      resolve();
      return;
    }

    if (not_added.includes(relativePath)) {
      await fs.rm(fullFilePath, { force: true });
      resolve();
      return;
    }

  } catch (error) {
    reject(error);
  }
});





const getFileDiff = (absoluteRepoPath, fullFilePath) => new Promise(async (resolve, reject) => {
  const gitRepo = simpleGit(absoluteRepoPath);

  try {
    // Convert full file path to path relative to the repo (required by Git CLI)
    const relativePath = path.relative(absoluteRepoPath, fullFilePath);

    // Get the current status of the repository
    const status = await gitRepo.status();

    if (status.not_added.includes(relativePath)) {
      // If the file is untracked, compare it with /dev/null to show the diff for new files
      const diffResult = await gitRepo.raw(['diff', '--no-index', '/dev/null', relativePath]);
      return resolve(diffResult);
    } else if (status.modified.includes(relativePath)) {
      // If the file is modified (tracked), show the diff of changes in the file
      const diffResult = await gitRepo.diff([relativePath]);
      return resolve(diffResult);
    } else if (status.deleted.includes(relativePath)) {
      // If the file is deleted, we need to show the diff for the staged deletion
      // First, let's stage the file again to view the diff
      await gitRepo.add([relativePath]);

      // Now show the diff for the file deletion (using --cached to show staged changes)
      const diffResult = await gitRepo.raw(['diff', '--cached', '--', relativePath]);
      console.log(diffResult);

      // After showing the diff, unstage it to keep things clean
      await gitRepo.reset(['--', relativePath]);

      return resolve(diffResult);
    }

  } catch (error) {
    reject(error);
  }
});




















const deleteMissingFiles = (workspaceFilesArray, validToken, t, filesList, workspaceId, requestId) => new Promise(
  async (resolve, reject) => {
    try {
      //create a mapping of file path to file id
      const filePathIdMapping = new Map(
        workspaceFilesArray.map(file => [file.full_file_path, file.id])
      );

      // Filter out the file paths that are missing in filesList
      const missingFileIds = Array.from(filePathIdMapping)
        .filter(([fullFilePath]) => !filesList.includes(fullFilePath))
        .map(([, id]) => id); // Extract only the IDs

      if (missingFileIds.length > 0) {

        // Update the to_date of the missing files
        await models.workspace_files.update(
          {
            modified_by_user_id: validToken.user_id,
            last_update_date: new Date(),
            to_date: new Date()
          },
          {
            where: {
              id: {
                [op.in]: missingFileIds,
                [op.ne]: workspaceFilesArray[0].id
              },
              workspace_id: workspaceId,
              tenant_id: validToken.tenant_id,
              to_date: {
                [op.gt]: new Date()
              }
            },
            transaction: t,
            logging: (query, elapsedTime) =>
              Utils.sequelizeLog({
                query,
                elapsedTime,
                requestId: requestId,
                serviceLogger: logger
              })
          });
      }
      resolve(true);
    }
    catch (error) {
      reject(error);
    }
  }
);



======================================================================

const discardGitFiles = ({ workspace_id, workspace_name, file_ids, file_paths, all }, request, response) => new Promise(
  async (resolve, reject) => {
    try {
      logger.info(`Discarding selected files in a Git Folder ...`, { requestId: request.requestId });

      const validToken = request.validToken;
      let discardFile;

      await new Utils.Tenant().validateTenantId(validToken.tenant_id, request.requestId).catch(e => {
        reject(Service.rejectResponse(
          {
            error_code: e.error_code,
            error_message: e.error || e
          },
          e.status || 500
        ));
        throw new Error(`Error while validating Tenant Id: ${e.error || e}`);
      });

      if (workspace_id === undefined && workspace_name === undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide any one of workspace_id or workspace_name to discard changes in a file within a Git folder.'
          },
          400
        ));
        return;
      }
      else if (workspace_id !== undefined && workspace_name !== undefined) {
        reject(Service.rejectResponse(
          {
            error: 'Please provide either workspace_id or workspace_name to discard changes in a file within a Git folder.'
          },
          400
        ));
        return;
      }

      const [workspace] = await checkWorkspaceExistsAndEnabled({
        workspace_id: workspace_id,
        workspace_name: workspace_name,
        workspace_permission_rfa: {
          permission: ["MANAGE", "EDIT", "RUN"],
          rfa: {
            code: "RFA-000070",
            message: `Insufficient permissions on the provided Workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `Name: '${workspace_name}'`} to discard changes in a file within a Git folder.`
          }
        },
        validToken,
        check_enable: undefined,
        request: request
      }).catch(error => {
        reject(Service.rejectResponse(
          {
            "error_code": error.error_code,
            "error_message": error.error_message || error
          },
          error.status_code || 500
        ));
        throw new Error(error.error_message || error);
      });

      if (all) {
        if (
          (!Array.isArray(file_ids) || file_ids.length === 0) &&
          (!Array.isArray(file_paths) || file_paths.length === 0)
        ) {
          reject(Service.rejectResponse(
            {
              error: 'Please provide at least one of file_ids or file_paths  to discard changes in a file within a Git folder.'
            },
            400
          ));
          return;
        } else if (
          Array.isArray(file_ids) && file_ids.length > 0 &&
          Array.isArray(file_paths) && file_paths.length > 0
        ) {
          reject(Service.rejectResponse(
            {
              error: 'Please provide either file_ids or file_paths, not both, to discard changes in a file within a Git folder.'
            },
            400
          ));
          return;
        }

        if ((Array.isArray(file_ids) && file_ids.length > 1) || (Array.isArray(file_paths) && file_paths.length > 1)) {
          reject(Service.rejectResponse(
            {
              error: 'When `all=true`, provide only one file_id or one file_path pointing to the Git Folder.'
            },
            400
          ));
          return;
        }

        const workspaceBasePath = `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`;
        const resolvedFilePaths = [];
        const fullFilePaths = [];
        const workspaceFiles = [];
        let workspaceFile = [];

        const isUsingFilePaths = Array.isArray(file_paths) && file_paths.length > 0;
        const listToProcess = isUsingFilePaths ? file_paths : file_ids;

        await config.sequelize.transaction(
          {
            logging: (query, elapsedTime) =>
              Utils.sequelizeLog({
                query,
                elapsedTime,
                requestId: request.requestId,
                serviceLogger: logger,
              }),
          },
          async (t) => {

            // start of for loop
            let file_paths = isUsingFilePaths ? listToProcess[0] : undefined;
            let file_ids = !isUsingFilePaths ? listToProcess[0] : undefined;

            if (file_paths) {
              [, file_paths] = await new WorkspaceFiles().resolvePath(
                file_paths,
                undefined,
                workspaceBasePath
              ).catch(error => {
                reject(Service.rejectResponse(
                  {
                    "error_code": error.error_code,
                    "error_message": error.error_message || error
                  },
                  error.status_code || 500
                ));
                throw new Error(error.error_message || error);
              });
            }

            workspaceFile = file_paths
              ? await sequelize.query(
                `SELECT id, lookup_file_types_id, lookup_git_providers_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(:filepath, NULL, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                {
                  replacements: {
                    filepath: file_paths,
                    _workspace_id: workspace[0].id,
                    _tenant_id: validToken.tenant_id,
                  },
                  type: sequelize.QueryTypes.SELECT,
                  transaction: t,
                  logging: (query, elapsedTime) =>
                    Utils.sequelizeLog({
                      query,
                      elapsedTime,
                      requestId: request.requestId,
                      serviceLogger: logger,
                    }),
                }
              )
              : await sequelize.query(
                `SELECT id, lookup_file_types_id, lookup_git_providers_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(NULL, :fileid, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                {
                  replacements: {
                    fileid: file_ids,
                    _workspace_id: workspace[0].id,
                    _tenant_id: validToken.tenant_id,
                  },
                  type: sequelize.QueryTypes.SELECT,
                  transaction: t,
                  logging: (query, elapsedTime) =>
                    Utils.sequelizeLog({
                      query,
                      elapsedTime,
                      requestId: request.requestId,
                      serviceLogger: logger,
                    }),
                }
              );

            if (!workspaceFile || workspaceFile.length < 1) {
              reject(Service.rejectResponse(
                {
                  error_code: "RFA-000206",
                  error_message: `The provided directory ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                },
                404
              ));
              throw new Error(`The provided directory ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
            }

            if (workspaceFile[0].lookup_file_types_id != global.lookupFileTypesByName['GIT_FOLDER']) {
              reject(Service.rejectResponse(
                {
                  error: `The provided  ${file_ids !== undefined ? `File ID: ${file_ids}` : `File path: '${file_paths}'`} is not a Git Folder in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                },
                400
              ));
              throw new Error(`The provided  ${file_ids !== undefined ? `File ID: ${file_ids}` : `File path: '${file_paths}'`} is not a Git Folder in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
            }

            // workspaceFiles.push(workspaceFile[0]);   this is not req here

            const filePath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
              replacements: {
                fileId: workspaceFile[0]?.id,
                workspaceId: workspace[0].id,
                tenantId: validToken.tenant_id,
              },
              transaction: t,
              logging: (query, elapsedTime) =>
                Utils.sequelizeLog({
                  query,
                  elapsedTime,
                  requestId: request.requestId,
                  serviceLogger: logger,
                }),
            });

            if (!filePath[0][0]) {
              reject(Service.rejectResponse(
                {
                  error_code: "RFA-000206",
                  error_message: `The provided file ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                },
                404
              ));
              throw new Error(`The provided file ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
            }

            const relativeFilePath = filePath[0]?.[0]?.path;
            // fullFilePaths.push(relativeFilePath);        this is not req here

            const filePathToDiscardFile = path.join(
              `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`,
              relativeFilePath
            );

            // resolvedFilePaths.push(filePathToDiscardFile);    this is also not required
            // end of for loop

            discardFile = await discardAllFiles(filePathToDiscardFile).catch(error => {
              reject(Service.rejectResponse(
                {
                  "error_code": error.error_code,
                  "error_message": error.error_message || error.message || error.error || error
                },
                error.status_code || 500
              ));
              throw new Error(error.error_message || error.message || error.error || error);
            });
          }
        );
      }
      else {

        if (
          (!Array.isArray(file_ids) || file_ids.length === 0) &&
          (!Array.isArray(file_paths) || file_paths.length === 0)
        ) {
          reject(Service.rejectResponse(
            {
              error: 'Please provide at least one of file_ids or file_paths  to discard changes in a file within a Git folder.'
            },
            400
          ));
          return;
        } else if (
          Array.isArray(file_ids) && file_ids.length > 0 &&
          Array.isArray(file_paths) && file_paths.length > 0
        ) {
          reject(Service.rejectResponse(
            {
              error: 'Please provide either file_ids or file_paths, not both, to discard changes in a file within a Git folder.'
            },
            400
          ));
          return;
        }

        const workspaceBasePath = `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`;
        const resolvedFilePaths = [];
        const fullFilePaths = [];
        const workspaceFiles = [];
        let workspaceFile = [];

        const isUsingFilePaths = Array.isArray(file_paths) && file_paths.length > 0;
        const listToProcess = isUsingFilePaths ? file_paths : file_ids;

        await config.sequelize.transaction(
          {
            logging: (query, elapsedTime) =>
              Utils.sequelizeLog({
                query,
                elapsedTime,
                requestId: request.requestId,
                serviceLogger: logger,
              }),
          },
          async (t) => {

            for (let i = 0; i < listToProcess.length; i++) {
              let file_paths = isUsingFilePaths ? listToProcess[i] : undefined;
              let file_ids = !isUsingFilePaths ? listToProcess[i] : undefined;

              if (file_paths) {
                [, file_paths] = await new WorkspaceFiles().resolvePath(
                  file_paths,
                  undefined,
                  workspaceBasePath
                ).catch(error => {
                  reject(Service.rejectResponse(
                    {
                      "error_code": error.error_code,
                      "error_message": error.error_message || error
                    },
                    error.status_code || 500
                  ));
                  throw new Error(error.error_message || error);
                });
              }

              workspaceFile = file_paths
                ? await sequelize.query(
                  `SELECT id, lookup_file_types_id, lookup_git_providers_id, git_repo_root_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(:filepath, NULL, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                  {
                    replacements: {
                      filepath: file_paths,
                      _workspace_id: workspace[0].id,
                      _tenant_id: validToken.tenant_id,
                    },
                    type: sequelize.QueryTypes.SELECT,
                    transaction: t,
                    logging: (query, elapsedTime) =>
                      Utils.sequelizeLog({
                        query,
                        elapsedTime,
                        requestId: request.requestId,
                        serviceLogger: logger,
                      }),
                  }
                )
                : await sequelize.query(
                  `SELECT id, lookup_file_types_id, lookup_git_providers_id, git_repo_root_id from YEEDU.GET_DIRECTORY_CONTENTS_WORKSPACE_FILES(NULL, :fileid, :_workspace_id, :_tenant_id, FALSE) LIMIT 1`,
                  {
                    replacements: {
                      fileid: file_ids,
                      _workspace_id: workspace[0].id,
                      _tenant_id: validToken.tenant_id,
                    },
                    type: sequelize.QueryTypes.SELECT,
                    transaction: t,
                    logging: (query, elapsedTime) =>
                      Utils.sequelizeLog({
                        query,
                        elapsedTime,
                        requestId: request.requestId,
                        serviceLogger: logger,
                      }),
                  }
                );

              if (!workspaceFile || workspaceFile.length < 1) {
                reject(Service.rejectResponse(
                  {
                    error_code: "RFA-000206",
                    error_message: `The provided file ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                  },
                  404
                ));
                throw new Error(`The provided file ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
              }

              if (
                workspaceFile[0].lookup_file_types_id === global.lookupFileTypesByName['GIT_FOLDER'] ||
                workspaceFile[0].lookup_file_types_id === global.lookupFileTypesByName['FOLDER']
              ) {
                reject(Service.rejectResponse(
                  {
                    error: `The provided ${file_ids !== undefined ? `File ID: ${file_ids}` : `File path: '${file_paths}'`} is not a file. Only files can be used to get the file-diff in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                  },
                  400
                ));
                throw new Error(`The provided ${file_ids !== undefined ? `File ID: ${file_ids}` : `File path: '${file_paths}'`} is not a file. Only files can be used to get the file-diff in the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
              }

              workspaceFiles.push(workspaceFile[0]);

              const filePath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
                replacements: {
                  fileId: workspaceFile[0]?.id,
                  workspaceId: workspace[0].id,
                  tenantId: validToken.tenant_id,
                },
                transaction: t,
                logging: (query, elapsedTime) =>
                  Utils.sequelizeLog({
                    query,
                    elapsedTime,
                    requestId: request.requestId,
                    serviceLogger: logger,
                  }),
              });

              if (!filePath[0][0]) {
                reject(Service.rejectResponse(
                  {
                    error_code: "RFA-000206",
                    error_message: `The provided file ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                  },
                  404
                ));
                throw new Error(`The provided file ${file_ids !== undefined ? `ID: ${file_ids}` : `path: '${file_paths}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
              }

              const relativeFilePath = filePath[0]?.[0]?.path;
              fullFilePaths.push(relativeFilePath);

              const filePathToDiscardFile = path.join(
                `${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`,
                relativeFilePath
              );

              resolvedFilePaths.push(filePathToDiscardFile);
            }

            const repoRootId = workspaceFile[0].git_repo_root_id;

            const repoPath = await sequelize.query(sqlToGetFilePathBasedonFileIdOfWorkspaceFiles, {
              replacements: {
                fileId: repoRootId,
                workspaceId: workspace[0].id,
                tenantId: validToken.tenant_id,
              },
              transaction: t,
              logging: (query, elapsedTime) =>
                Utils.sequelizeLog({
                  query,
                  elapsedTime,
                  requestId: request.requestId,
                  serviceLogger: logger,
                }),
            });

            if (!repoPath[0][0]) {
              reject(Service.rejectResponse(
                {
                  error_code: "RFA-000206",
                  error_message: `The provided directory ${repoRootId !== undefined ? `ID: ${repoRootId}` : `path: '${repoPath}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`
                },
                404
              ));
              throw new Error(`The provided directory ${repoRootId !== undefined ? `ID: ${repoRootId}` : `path: '${repoPath}'`} is not present or has been deleted within the specified workspace ${workspace_id !== undefined ? `ID: ${workspace_id}` : `name: '${workspace_name}'`}.`);
            }

            const filePathOfRepoRoot = path.join(`${config.YEEDU_SYSTEM_NFS_LOCAL_MOUNT_DIR}/yeedu/files/${validToken.tenant_id}/${workspace[0].id}`, repoPath[0]?.[0]?.path);

            discardFile = await discardMultipleFiles(filePathOfRepoRoot, resolvedFilePaths).catch(error => {
              reject(Service.rejectResponse(
                {
                  "error_code": error.error_code,
                  "error_message": error.error_message || error.message || error.error || error
                },
                error.status_code || 500
              ));
              throw new Error(error.error_message || error.message || error.error || error);
            });
          }
        );
      }
      logger.info(`Discarded selected files in a Git Folder.`, request.requestId);

      resolve(Service.successResponse(discardFile));
    } catch (e) {
      logger.error(e.stack || e.message || e.error || e, { requestId: request.requestId });
      reject(Service.rejectResponse(
        { error: e.message || e.stack || JSON.stringify(e) || e, requestId: request.requestId },
        e.status || 500
      ));
    }
  },
);







const discardGitFiles = async (request, response) => {
  await Controller.handleRequest(request, response, service.discardGitFiles);
};





const discardFileChanges = (absoluteRepoPath, fullFilePath) => new Promise(async (resolve, reject) => {
  const gitRepo = simpleGit(absoluteRepoPath);
  const relativePath = path.relative(absoluteRepoPath, fullFilePath);

  try {
    await fs.access(absoluteRepoPath); // Ensure repo path exists

    const status = await gitRepo.status();
    const { not_added, modified, deleted } = status;

    if (not_added.includes(relativePath)) {
      // Untracked file — just delete it
      await fs.rm(fullFilePath, { force: true });
      resolve();
      return;
    }

    if (modified.includes(relativePath) || deleted.includes(relativePath)) {
      // Modified or deleted tracked file — restore it
      await gitRepo.checkout(relativePath);
      resolve();
      return;
    }

    // No changes to discard
    resolve();

  } catch (error) {
    reject(error);
  }
});

const discardMultipleFiles = async (absoluteRepoPath, filePaths = []) => {
  try {
    // Runs all of them in parallel, and waits for all to finish — regardless of whether each one succeeds or fails.
    await Promise.allSettled(
      filePaths.map(fullFilePath => discardFileChanges(absoluteRepoPath, fullFilePath))
    );

    return {
      message: "Files discarded successfully."
    };

  } catch (err) {
    throw err;
  }
};

const discardAllFiles = (repoPathToDiscardAll) => new Promise(async (resolve, reject) => {
  const gitRepo = simpleGit(repoPathToDiscardAll);

  try {
    await fs.access(repoPathToDiscardAll); // Ensure repo path exists

    // Discard changes in tracked files
    await gitRepo.checkout('.');

    // Remove untracked files and directories (equivalent to `git clean -fd`)
    await gitRepo.raw(['clean', '-fd']);

    resolve({
      message: "Files discarded successfully."
    });

  } catch (error) {
    reject(error);
  }
});



  /workspace/files/git/discard:
    post:
      security:
        - bearerAuth: []
      description: |
        Discards changes for the provided files within the Git folder in the specified workspace.
      operationId: discardGitFiles
      parameters:
        - in: query
          name: workspace_id
          schema:
            type: integer
            format: int64
          description: Workspace ID used to locate the Git folder.
        - in: query
          name: workspace_name
          schema:
            type: string
          description: Workspace name used to locate the Git folder.
        - in: query
          name: file_ids
          schema:
            type: array
            items:
              type: integer
              format: int64
          description: List of file IDs to discard.
        - in: query
          name: file_paths
          schema:
            type: array
            items:
              type: string
              minLength: 1
          description: List of full file paths to discard.
        - in: query
          name: all
          schema:
            type: boolean
            default: false
          description: If set to true, discard changes for all files within the Git folder of the specified workspace. In this case, the user must provide **only one** entry in either `file_ids` or `file_paths`, which should point to the Git Folder.
      responses:
        "200":
          description: Discard result for the specified files..
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/DiscardFileResponse"
        "400":
          $ref: "#/components/responses/BadRequest"
        "401":
          $ref: "#/components/responses/Unauthenticated"
        "403":
          $ref: "#/components/responses/PermissionDenied"
        "404":
          $ref: "#/components/responses/NotFound"
        "500":
          $ref: "#/components/responses/Unknown"
      summary: Discard changes for one or more files in a Git workspace.
      tags:
        - Git
      x-swagger-router-controller: GitController
      x-eov-operation-handler: controllers/GitController







